# 🏃 Sprint Backlog - Devalign

**Duración del Proyecto:** 4 de Mayo de 2026 - 22 de Junio de 2026 (7 Semanas - 50 Días Totales)  
**Metodología:** Scrum (Iterativo e Incremental)  
**Sprints Planificados:** 4 Sprints  

---

## 📅 Resumen de Sprints

*   **Sprint 1:** 04/05/2026 – 16/05/2026 (13 Días)
*   **Sprint 2:** 17/05/2026 – 28/05/2026 (12 Días)
*   **Sprint 3:** 29/05/2026 – 09/06/2026 (12 Días)
*   **Sprint 4:** 10/06/2026 – 22/06/2026 (13 Días) - *Sprint Actual*

---

## 🏃 Sprint 1: Fundaciones, Scraper e Identidad
**Objetivo:** Establecer la infraestructura base, ingesta offline de datos de mercado con resiliencia avanzada y flujos de autenticación JIT mediante Supabase.

| ID(s) | Tarea / Incremento | Fecha de inicio | Fecha final | Días | Responsable | Estado |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- |
| TS-006, TS-027 | Configuración Base, Repositorios, Docker y Alembic | 04/05/2026 | 05/05/2026 | 2 | Dev | Completado |
| TS-009, TS-007 | Estrategia de Scraper, Pre-Filtrado y Rate Limiting | 06/05/2026 | 08/05/2026 | 3 | Dev | Completado |
| TS-012, TS-025, TS-002 | Resiliencia, Checkpoints y Carga Masiva | 09/05/2026 | 11/05/2026 | 3 | Dev | Completado |
| US01, US02, TS-024 | Auth Base: Registro, Login y Aprovisionamiento JIT | 12/05/2026 | 14/05/2026 | 3 | Dev | Completado |
| US05, TS-005 | OAuth y Middleware de Protección | 15/05/2026 | 16/05/2026 | 2 | Dev | Completado |

---

## 🏃 Sprint 2: Core de IA, Extracción y Clustering
**Objetivo:** Integrar la extracción estructurada mediante LLM y Voyage AI para embeddings, y entrenar el modelo de clústeres dinámicos.

| ID(s) | Tarea / Incremento | Fecha de inicio | Fecha final | Días | Responsable | Estado |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- |
| TS-003 | API de Extracción estructurada mediante LLM | 17/05/2026 | 19/05/2026 | 3 | Dev | Completado |
| TS-001 | Pipeline Normalización (Exact Match + Voyage embeddings) | 20/05/2026 | 21/05/2026 | 2 | Dev | Completado |
| TS-016 | Feature Engineering con Voyage AI | 22/05/2026 | 24/05/2026 | 3 | Dev | Completado |
| TS-015 | Análisis Exploratorio EDA | 25/05/2026 | 25/05/2026 | 1 | Dev | Completado |
| TS-017, TS-018 | Entrenamiento y Serialización de Clústeres (UMAP + HDBSCAN) | 26/05/2026 | 28/05/2026 | 3 | Dev | Completado |

---

## 🏃 Sprint 3: Motor de Similitud y Flujo del Usuario
**Objetivo:** Desarrollar el cálculo de afinidad técnica y construir las interfaces del usuario para carga (PDF/DOCX) y configuración de perfil.

| ID(s) | Tarea / Incremento | Fecha de inicio | Fecha final | Días | Responsable | Estado |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- |
| TS-010 | Integración pgvector y Motor Similitud (Weighted Jaccard) | 29/05/2026 | 31/05/2026 | 3 | Dev | Completado |
| US08, TS-020 | Flujo de Carga (PDF/DOCX) y Resiliencia | 01/06/2026 | 03/06/2026 | 3 | Dev | Completado |
| US09, TS-019 | UI/UX: Visualización y Persistencia de Habilidades del Perfil | 04/06/2026 | 07/06/2026 | 4 | Dev | Completado |
| US06, US07 | Aceptación de Políticas de Privacidad y Cierre de Sesión | 08/06/2026 | 09/06/2026 | 2 | Dev | Completado |

---

## 🏃 Sprint 4: Dashboard, Plan de Acción y Calidad (Sprint Activo)
**Objetivo:** Visualización avanzada de resultados (habilidades consolidadas vs brechas), entrega del plan de acción priorizado y robustecer la integración continua.

*Fecha Actual: 20 de Junio de 2026*

| ID(s) | Tarea / Incremento | Fecha de inicio | Fecha final | Días | Responsable | Estado |
| :--- | :--- | :--- | :--- | :---: | :--- | :--- |
| US10 | Dashboard Diagnóstico (Bento-Grid / Recharts) | 10/06/2026 | 13/06/2026 | 4 | Dev | Completado |
| US12 | Generación de Plan de Acción Determinista ($Prioridad = Peso \times Frecuencia$) | 14/06/2026 | 16/06/2026 | 3 | Dev | Completado |
| TS-008 | Logs de Auditoría y Monitoreo con Structlog | 17/06/2026 | 18/06/2026 | 2 | Dev | Completado |
| TS-014 | Pipeline de Integración Continua y Calidad (Ruff + Pytest) | 19/06/2026 | 20/06/2026 | 2 | Dev | Completado |
| TS-023 | Despliegue y Pruebas E2E Finales | 21/06/2026 | 22/06/2026 | 2 | Dev | Por Hacer |

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
- [📄 Documento de Requerimientos de Producto (PRD)](PRD.md)
- [📋 Product Backlog](PRODUCT_BACKLOG.md)
