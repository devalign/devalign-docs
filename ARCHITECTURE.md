# 🏗️ Arquitectura Técnica

Este documento describe la arquitectura técnica integral del sistema Devalign para el MVP. Detalla la topología de componentes, los flujos de datos síncronos y asíncronos, los módulos del ecosistema y las decisiones clave de infraestructura.

---

## Diagrama de Sistema

El sistema de Devalign se compone de cuatro módulos desacoplados y tres capas de infraestructura administrada.

```mermaid
graph TD
    subgraph Cliente ["Capa de Presentación (Next.js 16)"]
        Web[Aplicación Web Next.js 16 / React 19]
        SSR[Next.js Server-Side Rendering]
        Components[UI Modulares: Bento-Grid, Graph, Autocomplete]
    end

    subgraph Backend ["Capa de Negocio (FastAPI - Clean Architecture)"]
        API[API Endpoints /api/v1 - 23 Endpoints]
        Extractor[Zero-Waste LLM Extraction Engine]
        Normalizer[Skill Catalog & Taxonomy Engine - Lightcast/SFIA]
        Aligner[Weighted Jaccard Alignment Engine]
        SeniorityCalc[Dynamic Seniority & ICT Engine]
        ScraperAPI[Scraper Status Stub]
    end

    subgraph ML_Offline ["Módulos Offline y Procesamiento Batch"]
        Scraper[devalign-scraping<br>Multi-portal LATAM: Computrabajo, GetOnBoard, Remotive, WWR, Arbeitnow]
        ClusterPipe[devalign-ml<br>UMAP + HDBSCAN + Smooth IDF + Outlier Preservation]
    end

    subgraph Infraestructura ["Capa de Datos y Servicios (Supabase)"]
        Auth[Supabase Auth - JWT]
        Storage[Supabase Storage - CV Documents]
        DB[(PostgreSQL + pgvector 1024d<br>15 Tablas / 23 Migraciones Alembic)]
    end

    subgraph ML_External ["Modelos y APIs Externos"]
        Voyage[Voyage AI - voyage-4-lite 1024d]
        LLM[LLM Engine: Groq Llama 3.3 / OpenAI GPT-4o-mini]
    end

    Web -->|Autenticación JWT| Auth
    Web -->|Peticiones HTTP REST| API
    SSR -->|Fetch Data / SSR Cookies| API
    API -->|Validar JWT / RLS| DB
    API -->|Upload / Download CVs| Storage
    API -->|Embeddings Semánticos 1024d| Voyage
    API -->|Extracción Estructurada / Nombramiento| LLM
    API -->|Persistencia ORM SQLAlchemy| DB
    Scraper -->|Upsert Ofertas Normalizadas| DB
    ClusterPipe -->|Leer Ofertas + Embeddings| DB
    ClusterPipe -->|Escribir Clústeres + Centroides + Métricas| DB
```

---

## Flujos de Datos Core

### 1. Flujo Online (Procesamiento, Validación y Diagnóstico de CV)

El procesamiento de currículums opera bajo un patrón de dos fases desacopladas con puerta de validación humana para maximizar la precisión de las habilidades detectadas antes de consolidar el diagnóstico.

```mermaid
sequenceDiagram
    autonumber
    actor Usuario
    participant Web as Web (Next.js 16)
    participant API as API (FastAPI)
    participant Storage as Supabase Storage
    participant LLM as LLM Engine (Groq/OpenAI)
    participant Norm as Catalog & Taxonomy Engine
    participant DB as Postgres DB (pgvector)

    Usuario->>Web: Cargar CV (PDF/DOCX, máx 5MB)
    Web->>API: POST /api/v1/me/cv
    API->>Storage: Almacenar archivo binario original
    Storage-->>API: Ruta en storage (storage_path)
    API->>DB: Registrar CV (status: processing)
    API-->>Web: 201 Created (cv_id, status: processing)

    Note over API,LLM: Fase 1 - Extracción Estructurada Zero-Waste (Background Task)
    API->>API: Extraer texto crudo (pypdf / python-docx)
    API->>LLM: Prompt estructurado zero-waste (skills, exp, educación, rol)
    LLM-->>API: JSON estructurado
    API->>DB: Persistir extracted_data en cv_documents
    API->>DB: Actualizar status → skills_detected

    Note over Web,API: Cliente sondea cada 2s
    Web->>API: GET /api/v1/me/cvs/{cv_id}/status
    API-->>Web: {status: "skills_detected", extracted_skills: [...]}
    Web-->>Usuario: Presentar habilidades extraídas para revisión/edición

    Note over Usuario,API: Fase 2 - Finalización y Diagnóstico (Disparada por Usuario)
    Usuario->>Web: Confirmar/Editar habilidades (vía Autocomplete) y solicitar diagnóstico
    Web->>API: POST /api/v1/me/cv/{cv_id}/finalize (skills validadas)
    API-->>Web: 202 Accepted {status: "processing", message: "Diagnóstico en proceso..."}

    Note over API,DB: Ejecución Asíncrona de Fase 2 (Background Task)
    API->>Norm: Normalizar skills validadas contra catálogo Lightcast y aliases O(1)
    Norm->>DB: Consultar skills y skill_aliases
    DB-->>Norm: Canonical skills
    API->>Norm: Inferencia ascendente BFS (BELONGS_TO / REQUIRES)
    Norm->>DB: Consultar grafo de conocimiento (skill_relations)
    Norm-->>API: Skills expandidas con trazabilidad (inferred_from)
    API->>API: Calcular Weighted Jaccard vs clústeres activos (30% crédito dominio)
    API->>API: Calcular ICT Score (0-10) y Derivación Dinámica de Seniority
    API->>DB: Persistir profile_skills, diagnostics, diagnostic_skills
    API->>DB: Actualizar cv_documents status → completed y profiles.is_diagnosed → true

    Note over Web,API: Sondeo finaliza y carga perfil completo
    Web->>API: GET /api/v1/me
    API-->>Web: 200 OK (UserProfileDTO completo con diagnóstico, brechas e insights)
    Web-->>Usuario: Renderizar Dashboard interactivo con diagnóstico
```

### 2. Flujo Offline / Batch (Scraping Multi-Portal y Clustering LATAM)

```mermaid
graph TD
    subgraph Scraping ["devalign-scraping (Multi-Portal LATAM)"]
        CT[Computrabajo: PE, CO, CL, MX, AR<br>Playwright + BS4 + JSON-LD] --> Cleaner[Text Cleaner & Deduplication]
        GOB[GetOnBoard API REST] --> Cleaner
        REM[Remotive / WWR / Arbeitnow] --> Cleaner
        Cleaner --> ITFilter[Filtro IT Robusto de 2 Capas]
        ITFilter --> Struct[Extractor Estructurado: Salario, Experiencia, País]
        Struct --> Exporter[(DB: job_offers - Supabase Upsert)]
    end

    subgraph Clustering ["devalign-ml (Clustering & Enriquecimiento)"]
        DB2[(DB: job_offers + skills)] --> DataLoader[Data Loader & Embedding Matrix<br>Voyage AI 1024d]
        DataLoader --> UMAP[UMAP: 1024d → 15d]
        UMAP --> HDBSCAN[HDBSCAN min_cluster_size=15 + Smooth IDF]
        HDBSCAN --> Outliers[Preservación & Reasignación de Outliers]
        Outliers --> Enriched[Enriquecimiento Asíncrono LLM + Pydantic<br>Groq / OpenAI]
        Enriched --> Deploy[(DB: clusters + cluster_skills + offer_skills)]
    end

    Exporter --> DB2
```

---

## Componentes del Ecosistema

| Módulo | Repositorio | Tecnologías | Responsabilidad Core |
|---|---|---|---|
| **API Backend** | `devalign-api` | Python 3.12+, FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2, structlog | Capa de negocio central, endpoints REST, orquestación de LLMs, normalización semántica, inferencia de grafo, cálculo de Weighted Jaccard e ICT Score. |
| **Frontend Web** | `devalign-web` | Next.js 16 (App Router), React 19, Tailwind CSS v4, TypeScript, shadcn/ui, TanStack Query, Zustand, react-force-graph | Interfaz de usuario responsiva en español, visualización de diagnósticos Bento-Grid, radar de afinidades, autocompletado de habilidades y topología de mercado. |
| **Machine Learning** | `devalign-ml` | Python 3.12+, UMAP, HDBSCAN, scikit-learn, Voyage AI SDK, Groq/OpenAI, Pydantic | Pipeline no supervisado offline para reducción dimensional, agrupación de ofertas laborales, cálculo de centroides vectoriales 1024d y etiquetado de clústeres. |
| **Web Scraping** | `devalign-scraping` | Python 3.12+, Playwright, BeautifulSoup4, JSON-LD Parser, httpx | Extracción masiva de ofertas laborales IT en LATAM con control de concurrencia, filtros de relevancia tecnológica y exportación directa a Supabase. |
| **Persistencia & Auth** | `supabase` | PostgreSQL 15+, pgvector (1024d), Supabase Auth (JWT), Supabase Storage | Infraestructura DBaaS administrada, autenticación de desarrolladores y almacenamiento seguro de documentos CV. |

---

## Ambientes y Despliegue

| Ambiente | Capa | Plataforma | Tipo de Servicio | Dominio / URL |
|---|---|---|---|---|
| **Development** | Frontend | Localhost | Node.js (pnpm dev) | `http://localhost:3000` |
| **Development** | Backend | Localhost | Uvicorn (uv run) | `http://localhost:8000` |
| **Development** | Base de Datos | Supabase Cloud | PostgreSQL + pgvector | Instancia Supabase Dev |
| **Production** | Frontend | Vercel | Jamstack / Edge Serverless SSR | `https://devalign.vercel.app` |
| **Production** | Backend | Railway / Koyeb | Docker Container (multi-stage `uv`) | `https://api.devalign.com` |
| **Production** | Base de Datos | Supabase Cloud | Managed PostgreSQL + Storage | Instancia Supabase Prod |

---

## Decisiones Arquitectónicas

> **Decisión 1: SQLAlchemy y Alembic como SSOT Absoluto de la Base de Datos**
> Para garantizar reproducibilidad estricta y control de versiones del esquema relacional, **SQLAlchemy + Alembic** dentro de `devalign-api` actúa como la única fuente de verdad (Single Source of Truth) para la base de datos PostgreSQL. Se gestionan 23 migraciones formales. Supabase se utiliza como proveedor de infraestructura de nube (Postgres, Auth y Storage), evitando migraciones paralelas por Supabase CLI para prevenir desincronizaciones de esquema.

> **Decisión 2: Desacoplamiento Offline de Data Scraping y Clustering**
> La adquisición y el agrupamiento de ofertas de empleo operan de manera completamente desacoplada de la API en línea. `devalign-scraping` y `devalign-ml` se ejecutan como procesos batch y sincronizan datos directamente con PostgreSQL. La API expone un endpoint informativo `/api/v1/scraper/status` sin asumir sobrecarga computacional de scraping o entrenamiento en caliente.

> **Decisión 3: Ecosistema LLM Híbrido y Estabilidad Vectorial Unificada**
> Se mantiene una separación rigurosa entre tareas de generación semántica y operaciones vectoriales:
> 1. **Extracción y Nombramiento Semántico (LLMs):** Alterna dinámicamente según `APP_ENV`. En desarrollo utiliza **Groq** (`llama-3.3-70b-versatile`) para máxima velocidad y cero coste; en producción utiliza **OpenAI** (`gpt-4o-mini`) con structured outputs para garantizar formato JSON sin fisuras.
> 2. **Espacio Vectorial (Embeddings):** Utiliza **Voyage AI** (`voyage-4-lite`, 1024 dimensiones) de forma unificada en todos los entornos, garantizando la invariabilidad de las distancias matemáticas y umbrales de similitud de coseno en `pgvector`.

> **Decisión 4: Procesamiento de CV en Dos Fases con Puerta de Validación Humana**
> En lugar de ejecutar un análisis de caja negra directo a diagnóstico final, el flujo de CV divide el proceso en **Fase 1 (Extracción estructurada `skills_detected`)** y **Fase 2 (Normalización y Diagnóstico `completed`)**. Esto permite al usuario inspeccionar, añadir o retirar habilidades mediante autocompletado antes de calcular la afinidad y persistir brechas.

> **Decisión 5: Taxonomía Basada en Estándares Abiertos (Lightcast, SFIA y SWECOM)**
> El catálogo de habilidades adopta una estructura de gobernanza donde las habilidades conceptuales se mapean a la taxonomía **Lightcast Open Skills**, complementada por marcos SFIA 9 y SWECOM. Esto desacopla las habilidades técnicas accionables de los conceptos sombrilla y estandariza las categorías de mercado.

---

## 🔗 Referencias

- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
- [📄 Documento de Requerimientos de Producto (PRD)](PRD.md)
- [📋 Product Backlog](PRODUCT_BACKLOG.md)
- [🏃 Sprint Backlog](SPRINT_BACKLOG.md)
