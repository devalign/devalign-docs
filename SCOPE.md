# 🎯 Alcance MVP

Este documento delimita formalmente el alcance del Producto Mínimo Viable (MVP) de Devalign, definiendo las funcionalidades implementadas, las exclusiones técnicas explícitas y los criterios de aceptación (Definition of Done).

---

## Matriz de Control de Alcance (Features)

| Funcionalidad | ¿En MVP? | Fase de Entrega | Responsable | Notas y Justificación Técnica |
|---|:---:|:---:|---|---|
| **Carga de CV (PDF/DOCX)** | **Sí** | MVP | Frontend / Backend | Soporta archivos de hasta 5MB. Procesamiento asíncrono en 2 fases con Supabase Storage. |
| **Extracción Estructurada Zero-Waste** | **Sí** | MVP | Backend API | Extracción de bajo consumo de tokens con LLM (Groq en dev / OpenAI en prod). |
| **Validación Humana de Skills Extraídas** | **Sí** | MVP | Frontend / Backend | Flujo con estado `skills_detected` que permite al usuario revisar y editar habilidades antes del diagnóstico final. |
| **Gobernanza de Catálogo (Lightcast)** | **Sí** | MVP | Backend API | Habilidades conceptuales mapeadas a taxonomía Lightcast Open Skills, SFIA 9 y SWECOM. |
| **Normalización Semántica 3-Etapas** | **Sí** | MVP | Backend API | Coincidencia exacta $O(1)$ en aliases $\to$ Embeddings Voyage AI ($\ge 0.88$) $\to$ Fallback LLM. |
| **Inferencia Ascendente en Grafo** | **Sí** | MVP | Backend API | Recorrido BFS sobre aristas `BELONGS_TO` y `REQUIRES` con trazabilidad de proveniencia (`inferred_from`). |
| **Índice de Competencia (ICT Score)** | **Sí** | MVP | Backend API | Puntuación continua de $0.0$ a $10.0$ basada en evidencia de experiencia, proyectos y certificaciones. |
| **Derivación Dinámica de Seniority** | **Sí** | MVP | Backend API | Heurística híbrida por años de experiencia informados y palabras clave del perfil (SFIA 9). |
| **Alineación Weighted Jaccard** | **Sí** | MVP | Backend API | Jaccard ponderado con $30\%$ de crédito parcial por coincidencia de macro-dominios. |
| **Dashboard Bento-Grid Interactivo** | **Sí** | MVP | Frontend Web | Componentes modulares con radar de afinidad, métricas de mercado y visualización de fortalezas y brechas. |
| **Priorización de Brechas Técnicas** | **Sí** | MVP | Backend API | Clasificación automática en `critical`, `high` y `medium` basada en importancia y frecuencia de mercado. |
| **Edición Manual de Perfil y Skills** | **Sí** | MVP | Frontend / Backend | Endpoint `PUT /api/v1/me/skills` y `PATCH /api/v1/me` con recálculo dinámico en caliente. |
| **Autocompletado de Habilidades** | **Sí** | MVP | Frontend / Backend | Búsqueda rápida con ranking de coincidencia y despliegue de categorías de estándares (`SkillAutocomplete`). |
| **Topología Relacional de Mercado** | **Sí** | MVP | Frontend Web | Renderizado del grafo de habilidades en 2D/3D con `react-force-graph` y filtrado por clúster. |
| **Scraping Multi-Portal LATAM** | **Sí** | MVP | Scraping | Extracción multi-país en Computrabajo (PE, CO, CL, MX, AR), GetOnBoard, Remotive, WWR y Arbeitnow. |
| **Pipeline de Clustering Offline** | **Sí** | MVP | ML Engine | UMAP + HDBSCAN con Smooth IDF, preservación de outliers y nombrado LLM con validación Pydantic. |
| **Roadmaps de Estudio Paso a Paso** | **No** | Post-MVP | Backend / Frontend | Generación interactiva de planes de estudio detallados por tema (previsto para Fase 3). |
| **Seguimiento y Registro de Aprendizaje** | **No** | Post-MVP | Fullstack | Marcado paso a paso de cursos completados en base de datos. |
| **Portal B2B para Reclutadores** | **No** | Post-MVP | Fullstack | Búsqueda semántica y filtrado de candidatos para empresas. |
| **Scraping Autónomo Continuo en la Nube**| **No** | Post-MVP | Data Eng. | En MVP se orquesta por ejecuciones programadas y scripts batch. |

---

## Exclusiones Explícitas del MVP

1. **Generación Asíncrona de Roadmaps de Aprendizaje:** El MVP provee el diagnóstico de brechas técnicas priorizadas con insights accionables. La generación de planes de estudio paso a paso con itinerarios educativos detallados queda diferida como feature de retención Post-MVP.
2. **Plataforma de Reclutamiento y Monetización B2B:** El MVP se enfoca exclusivamente en la experiencia del desarrollador (B2C). Los filtros de búsqueda inversa para empresas y reclutadores quedan fuera del alcance actual.
3. **Scraping Autónomo en Tiempo Real por Streaming:** El scraper opera de forma batch desacoplada y vuelca la información directamente a PostgreSQL sin saturar los recursos de la API en producción.
4. **Pruebas de Carga E2E Masivas:** El conjunto de pruebas se concentra en pruebas unitarias y de regresión con cobertura superior al 80% en los módulos core.

---

## Criterios de Aceptación y Definition of Done (DoD)

Para considerar una funcionalidad finalizada dentro del MVP, debe cumplir con la totalidad de los siguientes criterios:

- [ ] **Cobertura de Pruebas Unitarias:** Cobertura de código $\ge 80\%$ comprobada mediante `pytest` en `devalign-api` y `devalign-scraping`.
- [ ] **Estricto Control de Tipos:** TypeScript estricto en `devalign-web` sin uso de `any` no justificado; Pydantic v2 y validadores estrictos en `devalign-api`.
- [ ] **Calidad de Código y Linting:** Aprobación sin advertencias de `ruff check` y `ruff format` en backend; `pnpm lint` en frontend.
- [ ] **Integridad de Esquema:** Toda modificación relacional debe estar respaldada por una migración formal de Alembic en `alembic/versions/`.
- [ ] **Documentación Sincronizada:** Los 6 documentos de la suite técnica en `docs/` deben reflejar con fidelidad el código fuente implementado.

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [📄 Documento de Requerimientos de Producto (PRD)](PRD.md)
- [📋 Product Backlog](PRODUCT_BACKLOG.md)
- [🏃 Sprint Backlog](SPRINT_BACKLOG.md)
