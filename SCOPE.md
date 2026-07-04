# 🎯 Alcance MVP

Este documento define los límites del Producto Mínimo Viable (MVP) de Devalign, especificando las características incluidas, las exclusiones del alcance y los criterios de aceptación.

---

## Matriz de Control de Alcance (Features)

| Funcionalidad | ¿En MVP? | Fase de Entrega | Responsable | Notas / Limitaciones técnicas |
| :--- | :---: | :---: | :--- | :--- |
| **Carga de CV (PDF/DOCX)** | **Sí** | Fase 1 (MVP) | Frontend / API | Límite 5MB. Procesamiento asíncrono en 2 fases (extracción + finalización). |
| **Extracción Estructurada de CV** | **Sí** | Fase 1 (MVP) | Backend API | LLM en segundo plano (Groq Llama 3.3 dev / OpenAI gpt-4o-mini prod). |
| **Normalización Semántica 3-etapas** | **Sí** | Fase 1 (MVP) | Backend API | O(1) aliases → Voyage AI cosine >= 0.88 → Fallback LLM. |
| **Inferencia Ascendente de Grafo** | **Sí** | Fase 1 (MVP) | Backend API | BFS sobre BELONGS_TO / REQUIRES / ALTERNATIVE_TO con inferred_from. |
| **ICT Score** | **Sí** | Fase 1 (MVP) | Backend API | Puntaje 0-10 (autodidacta + proyectos + experiencia + certificación). |
| **Seniority Estimation** | **Sí** | Fase 1 (MVP) | Backend API | Heurística por keywords y años (Junior/Mid/Senior). |
| **Alineación de Perfiles (Jaccard)** | **Sí** | Fase 1 (MVP) | Backend API | Weighted Jaccard modificado con 30% crédito parcial por dominio. |
| **Dashboard Bento-Grid** | **Sí** | Fase 1 (MVP) | Frontend Web | Responsivo con radar de afinidad, brechas priorizadas, Recharts. |
| **Priorización de Brechas** | **Sí** | Fase 1 (MVP) | Backend API | Prioridad = Peso × Frecuencia → critical / high / medium. |
| **Edición Manual de Perfil/Skills** | **Sí** | Fase 1 (MVP) | Frontend / API | Recálculo dinámico inmediato de afinidades y brechas. |
| **Historial de Documentos CV** | **Sí** | Fase 1 (MVP) | Frontend / API | Listar, borrar y re-analizar desde el panel. |
| **Topología del Mercado (Grafos)** | **Sí** | Fase 1 (MVP) | Frontend Web | react-force-graph 2D/3D del mapa relacional de skills. |
| **Evaluación por Especialidad** | **Sí** | Fase 1 (MVP) | Frontend / API | Evaluación voluntaria contra cualquier clúster del catálogo. |
| **Scraper Multi-Portal** | **Sí** | Fase 1 (MVP) | Scraping | Computrabajo (Playwright) + GetOnBoard (API REST). |
| **Pipeline de Clustering Offline** | **Sí** | Fase 1 (MVP) | ML | UMAP + HDBSCAN + Groq LLM naming + deploy a Supabase. |
| **Scraper Laboral Automatizado Cloud** | **No** | Fase 4 (Post-MVP) | Data Eng. | Ejecución manual local en MVP. |
| **Generación Interactiva de Roadmaps** | **No** | Fase 3 (Post-MVP) | Backend API | Guía de estudio paso a paso con LLM. Excluido del MVP. |
| **Dashboard B2B Reclutamiento** | **No** | Fase 4 (Post-MVP) | Fullstack | Búsqueda semántica de candidatos. |

---

## Exclusiones Explícitas (Fuera de Alcance del MVP)

1. **Sincronización Automatizada en Tiempo Real del Scraper:** No se implementará ejecución programada en la nube. El scraping se ejecuta manualmente de forma local y los datos se vuelcan a Supabase.
2. **Generación Síncrona / Interactiva de Roadmaps de Aprendizaje:** Planes de estudio paso a paso excluidos. El MVP provee brechas técnicas priorizadas como plan de acción.
3. **Planes de Reclutamiento B2B:** Dashboard y búsqueda semántica para reclutadores considerados Post-MVP.
4. **Pruebas de Integración:** El directorio `tests/integration/` existe pero está vacío. Las pruebas se limitan a unitarias (8 archivos en backend, 3 en scraping).

---

## Criterios de Aceptación y Definition of Done (DoD)

- **Cobertura de Pruebas:** Mínimo 80% de cobertura en pruebas unitarias del backend (`pytest`).
- **Linting y Calidad de Código:** Cero errores de `ruff check` / `ruff format` en backend; TypeScript estricto en frontend sin `any`.
- **Validación de Tipos y DTOs:** Pydantic v2 en backend, zod en frontend.
- **Versionado de Base de Datos:** Toda alteración del esquema debe contar con migración Alembic en `alembic/versions/`.

---

## Referencias

- [Arquitectura Técnica](ARCHITECTURE.md)
- [Contratos de Interfaz](CONTRACTS.md)
- [Modelo de Base de Datos](DATABASE.md)
- [Lógica Core e Inferencia](MODEL.md)
- [Roadmap de Producto](ROADMAP.md)
- [Documento de Requerimientos de Producto (PRD)](PRD.md)
- [Product Backlog](PRODUCT_BACKLOG.md)
- [Sprint Backlog](SPRINT_BACKLOG.md)
