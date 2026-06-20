# 🏗️ Arquitectura Técnica - Devalign

Este documento describe la arquitectura técnica del sistema Devalign para el MVP. Detalla la estructura de componentes, los flujos de datos síncronos y asíncronos, y las decisiones clave de infraestructura.

## 🗺️ Mapa de Arquitectura

El sistema de Devalign se compone de tres capas principales: la aplicación web en el cliente, la API de backend, y el proveedor de infraestructura Supabase.

```mermaid
graph TD
    subgraph Cliente ["Capa de Presentación (Next.js 16)"]
        Web[Aplicación Web]
        SSR[Next.js Server-Side Rendering]
    end

    subgraph Backend ["Capa de Negocio (FastAPI)"]
        API[API Endpoints]
        Extractor[LLM Extraction Engine]
        Normalizer[Skill Normalization Engine]
        Aligner[Alignment Engine]
    end

    subgraph Infraestructura ["Capa de Datos y Servicios (Supabase)"]
        Auth[Supabase Auth]
        Storage[Supabase Storage - CVs]
        DB[(PostgreSQL DB)]
    end

    subgraph ML_External ["Modelos y APIs Externos"]
        Voyage[Voyage AI API - Embeddings]
        LLM[LLM API - Extracción JSON]
    end

    %% Flujos de interacción
    Web -->|Autenticación| Auth
    Web -->|Peticiones HTTP| API
    SSR -->|Fetch Data / SSR Cookies| API
    API -->|Validar JWT / RLS| DB
    API -->|Upload / Download CVs| Storage
    API -->|Llamadas Semánticas| Voyage
    API -->|Extracción Estructurada| LLM
    API -->|Persistencia y Query| DB
```

> **Decisión de Arquitectura 1: SQLAlchemy/Alembic como SSOT de la Base de Datos**
> Para garantizar que el esquema relacional sea robusto, predecible y versionado, se ha determinado que **SQLAlchemy + Alembic** dentro de `devalign-api` actúe como el único origen de verdad (Single Source of Truth) para el esquema en PostgreSQL. 
> *Supabase* se utiliza exclusivamente como proveedor de infraestructura gestionada (Postgres, Storage, Auth). No se implementarán migraciones mediante Supabase CLI para evitar inconsistencias de esquema ("split-brain").

---

## 🔄 Flujos de Datos Core

### 1. Flujo Online (Procesamiento y Diagnóstico de CV)

Este flujo se ejecuta de manera síncrona/asíncrona cuando un usuario carga su CV en formato PDF o DOCX a través de la aplicación Web.

```mermaid
sequenceDiagram
    autonumber
    actor Usuario
    participant Web as Web (Next.js)
    participant API as API (FastAPI)
    participant Storage as Supabase Storage
    participant LLM as LLM Engine
    participant Norm as Normalization Engine
    participant DB as Postgres DB

    Usuario->>Web: Cargar CV (PDF/DOCX)
    Web->>API: POST /api/v1/users/me/cv
    API->>Storage: Guardar archivo original en storage
    Storage-->>API: URL de almacenamiento
    API->>API: Extraer texto del documento
    API->>LLM: Enviar texto para extracción estructurada (JSON)
    LLM-->>API: JSON con skills declaradas y metadatos
    loop Para cada skill extraída
        API->>Norm: Solicitar normalización semántica
        Norm->>DB: Buscar coincidencia exacta O(1)
        alt No hay coincidencia exacta
            Norm->>API: Generar embeddings (Voyage AI)
            Norm->>DB: Búsqueda vectorial (Cosine Similarity >= 0.88)
        end
        Norm-->>API: ID de skill estandarizada
    end
    API->>DB: Persistir datos de diagnóstico y relaciones
    API-->>Web: Confirmación de análisis exitoso
    Web->>API: GET /api/v1/profile/me
    API->>DB: Consultar diagnóstico y calcular Weighted Jaccard vs Clústeres
    DB-->>API: Retornar brechas y prioridades
    API-->>Web: Retornar JSON de Diagnóstico
    Web-->>Usuario: Mostrar panel con brechas y plan de acción
```

### 2. Flujo Offline / Batch (Entrenamiento y Agrupamiento)

Este flujo se ejecuta periódicamente de forma programada o manual mediante scripts de administración de datos en el backend para actualizar los clústeres del mercado.

```mermaid
graph TD
    A[Scraper Computrabajo] -->|Playwright + BS4| B[Ficheros CSV / Ofertas]
    B -->|Seed Script| C[(Base de Datos: Raw Jobs)]
    C -->|Batch Skill Normalizer| D[Normalización por Lotes: Embeddings Voyage AI]
    D -->|UMAP - Reducción a 15-d| E[Vector Space Reducido]
    E -->|HDBSCAN - min_cluster_size=15| F[Clústeres Identificados]
    F -->|Cálculo de Centroides| G[Persistencia de Centroides y Habilidades por Clúster en DB]
```

> **Decisión de Arquitectura 2: Integración de Scraper Puramente Offline**
> En el MVP, el raspado de vacantes de Computrabajo es un proceso desacoplado y fuera de línea. La API de backend no expone endpoints para disparar el scraping en tiempo real. Los datos recolectados se insertan directamente en la base de datos a través de scripts de inicialización (`seed`), eliminando la sobrecarga operativa en el servidor de producción.

---

## 🚀 Estrategia de Despliegue

La infraestructura del MVP está diseñada para ser altamente costo-efectiva, escalable y rápida de implementar utilizando plataformas PaaS modernas:

| Componente | Plataforma | Tipo de Servicio | Justificación |
| :--- | :--- | :--- | :--- |
| **Frontend Web** | Vercel | Jamstack / Serverless SSR | Excelente rendimiento en Next.js, caché global en CDN y fácil gestión de variables de entorno. |
| **API Backend** | Railway / Koyeb | Contenedor PaaS (Docker) | Despliegue nativo de FastAPI con auto-escalado simple y monitorización básica de CPU/RAM. |
| **Base de Datos & Auth** | Supabase | DBaaS (PostgreSQL + pgvector) | Postgres gestionado que incluye módulo vector para embeddings de 1024 dimensiones, manejo integrado de autenticación (JWT) y storage de archivos. |

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
