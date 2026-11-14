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

- **Inteligencia de Mercado (Offline):** Recolectar y procesar ofertas laborales de portales multi-país LATAM e internacionales (**Computrabajo** en PE, CO, CL, MX, AR; **GetOnBoard**, **Remotive**, **Arbeitnow**, **WeWorkRemotely**) mediante scraping local batch (`devalign-scraping`) con taxonomía sincronizada para alimentar el pipeline de clustering de mercado.
- **Profiling Inteligente (Online):** Procesar el CV del usuario (PDF/DOCX) en 2 fases: escaneo determinista 100% catálogo de skills en memoria (`scan_text_for_catalog_skills`) y síntesis de metadatos de perfil (`current_job_role`, `years_experience`, `professional_summary`) vía LLM liviano (300 tokens, 15s) en segundo plano.
- **Normalización de Habilidades 3-etapas:** Coincidencia exacta O(1) en aliases, búsqueda semántica vectorial (Voyage AI, cosine >= 0.88) y fallback LLM para skills no mapeadas.
- **Inferencia Ascendente de Habilidades:** Expandir competencias mediante arcos del grafo de conocimiento (`BELONGS_TO`, `REQUIRES`, `ALTERNATIVE_TO`) con trazabilidad (`inferred_from`).
- **ICT Score:** Puntaje de competencia unificado (0-10) con curva logarítmica de retornos decrecientes en experiencia (hasta 5.0 pts), proyectos personales (3.0 pts) y capacitación/certificaciones (cap 2.0 pts).
- **Seniority Estimation:** Heurística basada en keywords y años de experiencia (Junior/Mid/Senior/Staff).
- **Diagnóstico de Alineación:** Servicio de dominio desacoplado AffinityCalculator con Weighted Jaccard, radar de 7 dominios canónicos y filtrado por umbral de adyacencia (35%).
- **Dashboard Interactivo Bento-Grid:** Fortalezas, brechas priorizadas (`critical`, `high`, `medium`), afinidades secundarias, grafo de conocimiento interactivo 2D/3D.
- **Edición en Caliente:** Actualización manual de perfil y habilidades con recálculo dinámico.
- **Historial de Documentos CV:** Listar, eliminar y re-analizar currículos subidos.
- **Evaluación de Clústeres Custom:** Evaluación voluntaria contra cualquier especialidad del catálogo.
- **Pipeline de Clustering Offline:** UMAP (1024d → 15d) + HDBSCAN (min_cluster_size=15) con reasignación de ruido, nombrado vía Groq LLM.

---

## Arquitectura de Componentes

El sistema está implementado bajo una arquitectura desacoplada y modular con cuatro repositorios:

### Módulo de Ingesta y Scraping (`devalign-scraping`)
- **Portales:** Computrabajo multi-país (PE, CO, CL, MX, AR), GetOnBoard, Remotive, Arbeitnow y WeWorkRemotely
- **Arquitectura:** Segregación de parsers (`BrowserParser` con Playwright lazy-loaded vs `HttpParser` con `requests.Session` y reintentos automáticos), Strategy Pattern y SessionManager con checkpoints atómicos y auto-resume
- **Taxonomía en Ingesta:** `SkillTaxonomyService` sincronizado con la tabla `skills` de Supabase, caché local TTL de 7 días y respaldo offline empaquetado (`skills_taxonomy.json`)
- **Filtrado & Enriquecimiento:** Filtro IT robusto de 2 capas, extracción estructurada (salarios multi-moneda LATAM normalizados a USD, experiencia, país) y habilidades técnicas/blandas tipadas
- **Salida:** Exportación por lotes de 100 ofertas a Supabase con retención de checkpoints ante fallos de conexión o exportación a JSON local

### Módulo de Inteligencia y Servicio API (`devalign-api`)
- **Arquitectura:** Clean Architecture (Delivery, ML Engine, Scraper, Shared) con servicio de dominio puro `AffinityCalculator` y repositorio deduplicado
- **LLM:** Groq (Llama 3.3 70B) en desarrollo, OpenAI (gpt-4o-mini) en producción para síntesis de metadatos de perfil (300 tokens, 15s)
- **Embeddings:** Voyage AI (`voyage-4-lite`) unificado en todos los entornos
- **Normalización:** Aliases O(1) → Voyage vector search → LLM fallback
- **Inferencia:** BFS en grafo de conocimiento (BELONGS_TO / REQUIRES / ALTERNATIVE_TO)
- **Afinidad:** Weighted Jaccard con radar canónico de 7 dominios y filtrado por umbral de adyacencia
- **ICT Score:** Fórmula unificada en entidad de dominio con escala de retornos decrecientes
- **Seguridad e Infraestructura:** Verificación criptográfica JWT (HS256) obligatoria, Dockerfile non-root reproducible con `uv.lock`
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
- **Scraping:** Playwright, BeautifulSoup4, requests, lxml
- **Infraestructura:** Docker, Vercel (frontend), Railway/Koyeb (API)

---

## Referencias

- [Arquitectura Técnica](ARCHITECTURE.md)
- [Contratos de Interfaz](CONTRACTS.md)
- [Modelo de Base de Datos](DATABASE.md)
- [Lógica Core e Inferencia](MODEL.md)
- [Roadmap de Producto](ROADMAP.md)
- [Alcance MVP](SCOPE.md)
