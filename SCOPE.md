# 🎯 Alcance MVP - Devalign

Este documento define los límites del Producto Mínimo Viable (MVP) de Devalign, especificando detalladamente las características incluidas, las exclusiones del alcance y los criterios de aceptación para el desarrollo.

---

## 📊 Matriz de Control de Alcance (Features)

La siguiente tabla resume el alcance de las funcionalidades, distinguiendo estrictamente entre el MVP y el Alcance Futuro (Post-MVP):

| Funcionalidad | ¿En MVP? | Fase de Entrega | Responsable (Owner) | Notas / Limitaciones técnicas |
| :--- | :---: | :---: | :--- | :--- |
| **Carga de CV (PDF/DOCX)** | **Sí** | Fase 1 | Frontend / API | Límite máximo de 5MB por archivo cargado. |
| **Extracción Estructurada de Texto** | **Sí** | Fase 1 | Backend API | Uso de LLM offline para extraer JSON estructurado del CV. |
| **Normalización Semántica** | **Sí** | Fase 1 | Backend API | Método híbrido: Match exacto O(1) y similitud de coseno en pgvector $\ge 0.88$. |
| **Alineación de Perfiles** | **Sí** | Fase 1 | Backend API | Algoritmo Weighted Jaccard con 30% de peso por coincidencia parcial de dominios. |
| **Panel de Diagnóstico Técnico** | **Sí** | Fase 1 | Frontend Web | Interfaz web responsiva con gráficos simples de habilidades y brechas. |
| **Priorización de Brechas** | **Sí** | Fase 1 | Backend API | Cálculo determinista básico ($Prioridad = Peso \times Frecuencia$). |
| **Edición Manual de Perfil** | **Sí** | Fase 2 (Post-MVP) | Frontend / API | Modificación directa en panel para recalcular sin volver a subir CV. |
| **Motor FP-Growth** | **No** | Fase 3 (Post-MVP) | Data Engineer | Reemplazado en el MVP por el agrupamiento estático de UMAP + HDBSCAN. |
| **Generación Interactiva de Roadmaps** | **No** | Fase 3 (Post-MVP) | Backend API | Plan interactivo asíncrono y guardado de pasos. Queda excluido del MVP. |
| **Sincronización del Scraper en Tiempo Real** | **No** | Fase 4 (Post-MVP) | Data Engineer | El scraper se ejecuta exclusivamente de forma manual y local (offline batch import). |

---

## 🚫 Exclusiones Explícitas (Fuera de Alcance del MVP)

Para asegurar la viabilidad del lanzamiento técnico, se excluyen explícitamente los siguientes componentes:

1. **Sincronización Automatizada en Tiempo Real del Scraper:** No se implementará comunicación directa del scraper con la API en caliente. El mercado se actualiza mediante un proceso manual offline que vuelca información consolidada en un CSV y luego ejecuta scripts de inicialización (`seed`).
2. **Generación Síncrona de Roadmaps mediante LLM:** Los planes interactivos de estudio están totalmente fuera del alcance. El MVP proveerá recomendaciones genéricas basadas en las prioridades de habilidades sin interactividad ni persistencia de pasos de aprendizaje.
3. **Reglas de Asociación Avanzadas (FP-Growth):** No se utilizará minería de datos compleja para buscar patrones de co-ocurrencia. Toda la afinidad se calcula en base a la distancia matemática de los centroides generados por HDBSCAN y la pertenencia a clústeres.

---

## 🏆 Criterios de Aceptación y Definition of Done (DoD)

Para certificar que una funcionalidad ha sido completada y está lista para producción en el MVP, debe cumplir con los siguientes estándares:

* **Cobertura de Pruebas:** Al menos el 80% de cobertura de código en las pruebas unitarias del backend (`pytest`).
* **Linting y Calidad de Código:** Ningún error reportado por `ruff check` en la suite de backend, y TypeScript estricto habilitado en frontend sin advertencias de tipos (`any`).
* **Validación de Tipos:** Estricto uso de Pydantic para el análisis sintáctico de las cargas de datos en la API.
* **Flujo de Base de Datos:** Cualquier alteración del esquema relacional debe contar con su correspondiente archivo de migración de Alembic en `alembic/versions`.

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
