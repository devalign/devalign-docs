# 📄 Documento de Requerimientos de Producto (PRD)

Este documento define los requisitos funcionales, el alcance técnico y las decisiones de producto para el MVP de Devalign, reflejando el estado actual de los repositorios del proyecto.

---

## Título Propuesto

**Aplicación web con motor de inferencia basado en Machine Learning para el análisis de competencias técnicas demandadas por el sector IT orientado a desarrolladores del Perú.**

---

## Problemática y Alcance

### Problema
Existe una asimetría de información crítica entre la formación técnica de los desarrolladores y la demanda real del sector IT. El problema no es solo la falta de experiencia, sino la **desalineación técnica**: los desarrolladores enfrentan una sobrecarga de información y carecen de una ruta clara para especializarse en los stacks tecnológicos que el mercado realmente demanda y valora.

### Propósito
Desarrollar un sistema inteligente que permita identificar automáticamente la especialidad técnica de un desarrollador, diagnosticar su nivel de alineación con el mercado laboral y priorizar el cierre de sus brechas técnicas.

### Alcance del MVP

- **Inteligencia de Mercado (Offline):** Recolectar y procesar ofertas laborales de **Computrabajo** (Playwright + BS4) y **GetOnBoard** (API REST) mediante scraping local (`devalign-scraping`) para descubrir clústeres tecnológicos vivos en el mercado.
- **Profiling Inteligente (Online):** Procesar el CV del usuario (PDF/DOCX) mediante LLMs en segundo plano (Groq Llama 3.3 en desarrollo / OpenAI gpt-4o-mini en producción) para extraer su perfil, experiencia laboral, educación e identificar sus competencias iniciales.
- **Normalización de Habilidades 3-etapas:** Coincidencia exacta O(1) en aliases, búsqueda semántica vectorial (Voyage AI, cosine >= 0.88) y fallback LLM para skills no mapeadas.
- **Inferencia Ascendente de Habilidades:** Expandir competencias mediante arcos del grafo de conocimiento (`BELONGS_TO`, `REQUIRES`, `ALTERNATIVE_TO`) con trazabilidad (`inferred_from`).
- **ICT Score:** Puntaje de competencia (0-10) basado en autodidacta, proyectos personales, años de experiencia y certificaciones.
- **Seniority Estimation:** Heurística basada en keywords y años de experiencia (Junior/Mid/Senior/Staff).
- **Diagnóstico de Alineación:** Weighted Jaccard Modificado con 30% de crédito parcial por coincidencia de dominio.
- **Dashboard Interactivo Bento-Grid:** Fortalezas, brechas priorizadas (`critical`, `high`, `medium`), afinidades secundarias, grafo de conocimiento interactivo 2D/3D.
- **Edición en Caliente:** Actualización manual de perfil y habilidades con recálculo dinámico.
- **Historial de Documentos CV:** Listar, eliminar y re-analizar currículos subidos.
- **Evaluación de Clústeres Custom:** Evaluación voluntaria contra cualquier especialidad del catálogo.
- **Pipeline de Clustering Offline:** UMAP (1024d → 15d) + HDBSCAN (min_cluster_size=15) con reasignación de ruido, nombrado vía Groq LLM.

---

## Arquitectura de Componentes

El sistema está implementado bajo una arquitectura desacoplada y modular con cuatro repositorios:

### Módulo de Ingesta y Scraping (`devalign-scraping`)
- **Portales:** Computrabajo Perú (Playwright + BeautifulSoup) y GetOnBoard (API REST)
- **Estrategia:** Strategy Pattern, Rate Limiting, Checkpoints, Auto-Resume
- **Filtrado:** Dos capas de filtrado IT (título/URL y análisis profundo)
- **Salida:** Exportación a Supabase (upsert) o JSON local

### Módulo de Inteligencia y Servicio API (`devalign-api`)
- **Arquitectura:** Clean Architecture (Delivery, ML Engine, Scraper, Shared)
- **LLM:** Groq (Llama 3.3 70B) en desarrollo, OpenAI (gpt-4o-mini) en producción
- **Embeddings:** Voyage AI (`voyage-4-lite`) unificado en todos los entornos
- **Normalización:** Aliases O(1) → Voyage vector search → LLM fallback
- **Inferencia:** BFS en grafo de conocimiento (BELONGS_TO / REQUIRES / ALTERNATIVE_TO)
- **Afinidad:** Weighted Jaccard con 30% de crédito parcial por dominio
- **Base de Datos:** SQLAlchemy + Alembic como SSOT, PostgreSQL + pgvector

### Módulo de Machine Learning Offline (`devalign-ml`)
- **Pipeline:** Media-pooling de embeddings 1024d → UMAP (15d) → HDBSCAN → Reasignación → Centroides
- **Nombrado:** Groq LLM para etiquetar clústeres semánticamente
- **Evaluación:** Silhouette Score (excluyendo ruido)

### Módulo Cliente Web (`devalign-web`)
- **Framework:** Next.js 16 App Router, React 19, Tailwind CSS v4
- **Visualizaciones:** Recharts, react-force-graph 2D/3D, Bento-grid
- **Auth:** Supabase SSR (email/password + Google OAuth)
- **Estado:** TanStack Query + Zustand

---

## Tecnologías Utilizadas

- **Backend:** FastAPI (Python 3.12+), SQLAlchemy + Alembic, Pydantic v2
- **Base de Datos & Auth:** Supabase (PostgreSQL, pgvector 1024-d, Storage, Auth)
- **ML / Analítica:** UMAP + HDBSCAN (offline), Voyage AI embeddings
- **LLM:** Groq API (Llama 3.3 70B) / OpenAI API (gpt-4o-mini)
- **Frontend:** Next.js 16, React 19, Tailwind CSS v4, shadcn/ui, Framer Motion
- **Scraping:** Playwright, BeautifulSoup, lxml
- **Infraestructura:** Docker, Vercel (frontend), Railway/Koyeb (API)

---

## Referencias

- [Arquitectura Técnica](ARCHITECTURE.md)
- [Contratos de Interfaz](CONTRACTS.md)
- [Modelo de Base de Datos](DATABASE.md)
- [Lógica Core e Inferencia](MODEL.md)
- [Roadmap de Producto](ROADMAP.md)
- [Alcance MVP](SCOPE.md)
- [Product Backlog](PRODUCT_BACKLOG.md)
- [Sprint Backlog](SPRINT_BACKLOG.md)
