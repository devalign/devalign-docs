# 🏗️ Arquitectura Técnica

Este documento describe la arquitectura técnica del sistema Devalign para el MVP. Detalla la estructura de componentes, los flujos de datos síncronos y asíncronos, y las decisiones clave de infraestructura.

## Mapa de Arquitectura

El sistema de Devalign se compone de cuatro módulos principales y tres capas de infraestructura.

```mermaid
graph TD
    subgraph Cliente ["Capa de Presentación (Next.js 16)"]
        Web[Aplicación Web]
        SSR[Next.js Server-Side Rendering]
    end

    subgraph Backend ["Capa de Negocio (FastAPI)"]
        API[API Endpoints /api/v1]
        Extractor[LLM Extraction Engine]
        Normalizer[Skill Normalization Engine]
        Aligner[Alignment Engine]
        ScraperAPI[Scraper Status Stub]
    end

    subgraph ML_Offline ["Módulos Offline"]
        Scraper[devalign-scraping<br>Playwright + BS4]
        ClusterPipe[devalign-ml<br>UMAP + HDBSCAN]
    end

    subgraph Infraestructura ["Capa de Datos y Servicios (Supabase)"]
        Auth[Supabase Auth]
        Storage[Supabase Storage - CVs]
        DB[(PostgreSQL + pgvector)]
    end

    subgraph ML_External ["Modelos y APIs Externos"]
        Voyage[Voyage AI - Embeddings]
        LLM[LLM API - Extracción JSON<br>Groq / OpenAI]
    end

    Web -->|Autenticación| Auth
    Web -->|Peticiones HTTP| API
    SSR -->|Fetch Data / SSR Cookies| API
    API -->|Validar JWT / RLS| DB
    API -->|Upload / Download CVs| Storage
    API -->|Llamadas Semánticas| Voyage
    API -->|Extracción Estructurada| LLM
    API -->|Persistencia y Query| DB
    Scraper -->|Upsert Ofertas| DB
    ClusterPipe -->|Leer Skills + Ofertas| DB
    ClusterPipe -->|Escribir Clústeres + Centroides| DB
```

> **Decisión de Arquitectura 1: SQLAlchemy/Alembic como SSOT de la Base de Datos**
> Para garantizar que el esquema relacional sea robusto, predecible y versionado, se ha determinado que **SQLAlchemy + Alembic** dentro de `devalign-api` actúe como el único origen de verdad (Single Source of Truth) para el esquema en PostgreSQL.
> *Supabase* se utiliza exclusivamente como proveedor de infraestructura gestionada (Postgres, Storage, Auth). No se implementarán migraciones mediante Supabase CLI para evitar inconsistencias de esquema ("split-brain").

---

## Flujos de Datos Core

### 1. Flujo Online (Procesamiento y Diagnóstico de CV)

Este flujo se ejecuta cuando un usuario carga su CV en formato PDF o DOCX a través de la aplicación Web. El procesamiento ocurre en dos fases asíncronas.

```mermaid
sequenceDiagram
    autonumber
    actor Usuario
    participant Web as Web (Next.js 16)
    participant API as API (FastAPI)
    participant Storage as Supabase Storage
    participant LLM as LLM Engine (Groq/OpenAI)
    participant Norm as Normalization & Inference
    participant DB as Postgres DB

    Usuario->>Web: Cargar CV (PDF/DOCX)
    Web->>API: POST /api/v1/me/cv
    API->>Storage: Guardar archivo original
    Storage-->>API: Ruta de almacenamiento
    API->>DB: Registrar CV (status: processing)
    API-->>Web: 201 Created (cv_id)

    Note over API,DB: Fase 1 - Extracción (Background Task)
    API->>API: Extraer texto del documento
    API->>LLM: Enviar texto para extracción estructurada (JSON)
    LLM-->>API: JSON con skills, experiencia, educación, cargo
    API->>DB: Persistir extracted_data en cv_documents
    API->>DB: Actualizar status → extracted

    Note over Web,API: Cliente sondea cada 3s
    Web->>API: GET /api/v1/me/cv/status
    API-->>Web: {status: "extracted", cv_id}

    Note over API,DB: Fase 2 - Finalización (disparada por usuario)
    Usuario->>Web: Validar y finalizar análisis
    Web->>API: POST /api/v1/me/cv/{cv_id}/finalize
    API->>Norm: Normalizar skills extraídas
    Note over Norm: 1. O(1) exact match en skill_aliases<br>2. Voyage AI embedding + cosine >= 0.88<br>3. Fallback LLM para skills no mapeadas
    Norm->>DB: Consultar skill_aliases y skills
    DB-->>Norm: Skills canónicas
    API->>Norm: Inferencia ascendente (BELONGS_TO / REQUIRES)
    Norm->>DB: Consultar grafo de conocimiento
    Norm-->>API: Skills expandidas con inferred_from
    API->>API: Calcular Weighted Jaccard vs clústeres
    API->>API: Calcular ICT Score y Seniority
    API->>DB: Persistir profile_skills, diagnostics, diagnostic_skills
    API->>Web: 200 OK (UserProfileDTO completo)
    Web-->>Usuario: Dashboard con diagnóstico y brechas
```

### 2. Flujo Offline / Batch (Scraping y Clustering)

```mermaid
graph TD
    subgraph Scraping ["devalign-scraping"]
        CT[Computrabajo<br>Playwright + BS4] -->|Estrategia Pattern| Parser1[ComputrabajoParser]
        GOB[GetOnBoard<br>API REST] -->|Parser Directo| Parser2[GetOnBoardParser]
        Parser1 --> Clean[Text Cleaner]
        Parser2 --> Clean
        Clean --> Filter[IT Job Filter<br>2-capas]
        Filter --> Export[(DB: job_offers)]
        Filter -->|--no-supabase| JSON[JSON Local]
    end

    subgraph Clustering ["devalign-ml"]
        DB2[(DB: skills + job_offers)] --> Load[Data Loader<br>mean-pool embeddings 1024d]
        Load --> UMAP[UMAP: 1024d → 15d]
        UMAP --> HDBSCAN[HDBSCAN min_cluster_size=15]
        HDBSCAN --> Eval[Silhouette Score]
        Eval --> Reassign[Reasignar Ruido a centroide más cercano]
        Reassign --> Name[Groq LLM: nombrar clústeres]
        Name --> Deploy[(DB: clusters + cluster_skills)]
    end

    Export --> DB2
```

> **Decisión de Arquitectura 2: Integración de Scraper Puramente Offline**
> En el MVP, el raspado de vacantes es un proceso desacoplado y fuera de línea. La API de backend expone un endpoint `/api/v1/scraper/status` como stub informativo. Los datos recolectados por `devalign-scraping` se insertan directamente en la base de datos a través del exportador Supabase, eliminando la sobrecarga operativa en el servidor de producción.

> **Decisión de Arquitectura 3: Ecosistema LLM Híbrido y Estabilidad Vectorial**
> El sistema divide las tareas de IA en dos servicios desacoplados:
> 1. **Extracción Semántica (LLMs):** Alterna dinámicamente según `APP_ENV`. En desarrollo, utiliza **Groq** (Llama 3.3 70B) para latencia mínima y cero costes. En producción, utiliza **OpenAI** (gpt-4o-mini) con Structured Outputs para consistencia en el formateo JSON.
> 2. **Normalización y Búsqueda (Embeddings):** Utiliza **Voyage AI** (`voyage-4-lite`) de forma unificada en desarrollo y producción para salvaguardar la estabilidad matemática de los vectores almacenados en PostgreSQL (`pgvector`).

---

## Estrategia de Despliegue

| Componente | Plataforma | Tipo de Servicio | Justificación |
| :--- | :--- | :--- | :--- |
| **Frontend Web** | Vercel | Jamstack / Serverless SSR | Excelente rendimiento en Next.js, caché global en CDN y fácil gestión de variables de entorno. |
| **API Backend** | Railway / Koyeb | Contenedor PaaS (Docker) | Despliegue nativo de FastAPI con auto-escalado simple y monitorización básica de CPU/RAM. |
| **Base de Datos & Auth** | Supabase | DBaaS (PostgreSQL + pgvector) | Postgres gestionado que incluye módulo vector para embeddings de 1024 dimensiones, manejo integrado de autenticación (JWT) y storage de archivos. |

El backend utiliza un Dockerfile multi-stage: builder con `uv` para instalar dependencias y runtime que copia el `.venv` + código fuente. Expone el puerto 8000.

---

## Módulos del Sistema

### devalign-scraping
- Scraper multi-portal: **Computrabajo** (Playwright + BeautifulSoup) y **GetOnBoard** (API REST)
- Patrón Strategy para extensibilidad a nuevos portales
- Rate limiting (2.5-5s Computrabajo, 0.5-1.5s GetOnBoard)
- Checkpoints y auto-resume en interrupción
- Filtro IT de dos capas (pre-filtro título/URL, post-filtro profundo)
- Exportación a Supabase (upsert) o JSON local

### devalign-ml
- Pipeline offline de clustering no supervisado
- Media-pooling de embeddings Voyage AI (1024d) por oferta
- Reducción UMAP (1024d → 15d) + HDBSCAN (min_cluster_size=15)
- Reasignación de ruido al centroide más cercano
- Nombrado de clústeres vía Groq LLM (llama-3.3-70b-versatile)
- Evaluación: Silhouette Score (excluyendo ruido)
- Modos: dry-run (train + eval + plot) y deploy (persistir en Supabase)

### devalign-api
- Arquitectura Clean Architecture con 18 endpoints REST
- Motores: CV parsing (pypdf, python-docx), LLM extraction, skill normalization, upward inference, Weighted Jaccard alignment, ICT scoring, seniority estimation
- 15 tablas SQLAlchemy, 20 migraciones Alembic
- Logging con structlog (pretty console en dev, JSON en prod)

### devalign-web
- Next.js 16 App Router, React 19, Tailwind CSS v4
- shadcn/ui (New York), Framer Motion, Recharts, react-force-graph
- Auth con Supabase SSR, datos con TanStack Query, estado con Zustand
- 6 páginas protegidas + login + landing + términos

---

## Referencias

- [Contratos de Interfaz](CONTRACTS.md)
- [Modelo de Base de Datos](DATABASE.md)
- [Lógica Core e Inferencia](MODEL.md)
- [Roadmap de Producto](ROADMAP.md)
- [Alcance MVP](SCOPE.md)
- [Documento de Requerimientos de Producto (PRD)](PRD.md)
- [Product Backlog](PRODUCT_BACKLOG.md)
- [Sprint Backlog](SPRINT_BACKLOG.md)
