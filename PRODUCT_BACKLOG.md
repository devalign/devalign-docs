# 📋 Product Backlog

Este documento contiene el Product Backlog de Devalign, alineado con el alcance real implementado en el sistema (Voyage AI, LLM de extracción, Supabase, pgvector, UMAP/HDBSCAN y mapa visual de grafos), organizado y categorizado mediante **Épicas (Epics)** para trazabilidad.

## 🗺️ Definición de Épicas

*   **EPIC-01: Ingesta de Datos de Mercado (Scraping)** - Desarrollo de la infraestructura para recolectar ofertas de trabajo IT del mercado (Computrabajo) mediante scraping offline.
*   **EPIC-02: Core de IA, Normalización y Similitud** - Motor de inteligencia artificial, normalización de habilidades usando Voyage AI, similitud mediante pgvector y clustering por UMAP + HDBSCAN.
*   **EPIC-03: Identidad y Seguridad** - Autenticación y control de accesos usando Supabase Auth.
*   **EPIC-04: Experiencia de Usuario, Diagnóstico y Perfil (Frontend/Backend)** - Flujos interactivos para subir el CV, editar perfiles y habilidades, evaluar clústeres, listar historial de CVs y mapa visual de grafos.
*   **EPIC-05: DevOps, Monitoreo y Calidad** - CI/CD, contenedorización, logging, migraciones de Alembic y auditoría.

---

## 📝 Detalle de Historias por Épica

| PASO | ÉPICA ID | ID TAREA | TÍTULO | DESCRIPCIÓN | SP | ESTADO |
| :---: | :--- | :--- | :--- | :--- | :---: | :--- |
| 1 | **EPIC-05** | TS-027 | Migraciones y Esquema de BD | Como Developer, quiero utilizar Alembic para versionar y aplicar los cambios del esquema de la base de datos relacional de forma de controlada. | 3 | Completado |
| 2 | **EPIC-01** | TS-009 | Estrategia de Scraper | Como Developer, quiero aplicar el patrón Strategy, para extraer ofertas de forma resiliente de Computrabajo. | 5 | Completado |
| 3 | **EPIC-01** | TS-012 | Resiliencia y Reintentos | Como Developer, quiero implementar Retry Policies, para asegurar tolerancia a fallos en el scraper. | 5 | Completado |
| 4 | **EPIC-01** | TS-025 | Checkpoints y Auto-Resume | Como Developer, quiero implementar un sistema de checkpoints y guardado de estado en el scraper, para evitar pérdida de datos en caso de corte. | 3 | Completado |
| 5 | **EPIC-01** | TS-007 | Rate Limiting Ético | Como Developer, quiero un controlador de frecuencia y delays aleatorios, para cumplir con la ética del scraping. | 3 | Completado |
| 6 | **EPIC-01** | TS-002 | Carga Masiva (Upsert) | Como Developer, quiero lógica bulk insert optimizada en Python, para actualizar la DB de mercado. | 5 | Completado |
| 7 | **EPIC-02** | TS-001 | Pipeline de Normalización | Como Developer, quiero transformar texto de habilidades usando Exact Match O(1) y fallback semántico (embeddings de Voyage AI $\ge 0.88$). | 5 | Completado |
| 8 | **EPIC-02** | TS-003 | API de Extracción | Como Developer, quiero un endpoint para extraer JSON estructurado de habilidades usando el SDK de LLM (Groq/OpenAI). | 8 | Completado |
| 9 | **EPIC-02** | TS-015 | Análisis Exploratorio | Como Developer, quiero realizar EDA, para identificar patrones de demanda IT. | 5 | Completado |
| 10 | **EPIC-02** | TS-016 | Feature Engineering | Como Developer, quiero transformar variables con embeddings de Voyage AI para clustering. | 5 | Completado |
| 11 | **EPIC-02** | TS-017 | Entrenar UMAP + HDBSCAN | Como Developer, quiero entrenar el modelo de agrupamiento reduciendo a 15-d con UMAP y agrupando con HDBSCAN (min_cluster_size=15), para definir especialidades IT de forma dinámica. | 8 | Completado |
| 12 | **EPIC-02** | TS-018 | Serialización del Modelo | Como Developer, quiero exportar los centroides y etiquetas del modelo para integrarlos en el backend. | 3 | Completado |
| 13 | **EPIC-02** | TS-010 | Motor de Similitud | Como Developer, quiero comparar el perfil usando Jaccard Ponderado (Weighted Jaccard) con 30% por coincidencia parcial. | 8 | Completado |
| 14 | **EPIC-03** | US01 | Registro de Usuario | Como usuario, quiero registrarme usando mi correo y contraseña integrado con Supabase Auth. | 3 | Completado |
| 15 | **EPIC-03** | US02 | Inicio de sesión | Como usuario, quiero autenticarme mediante Supabase Auth, para acceder a la plataforma. | 3 | Completado |
| 16 | **EPIC-03** | US05 | Login con Google | Como usuario, quiero autenticarme mediante mi cuenta de Google (OAuth), para acceder rápido. | 3 | Completado |
| 17 | **EPIC-03** | US07 | Cierre de Sesión | Como usuario, quiero cerrar mi sesión activa, para garantizar la seguridad. | 1 | Completado |
| 18 | **EPIC-03** | TS-005 | Middleware Protección | Como Developer, quiero proteger rutas para restringir el acceso a endpoints sensibles (JWT validado). | 3 | Completado |
| 19 | **EPIC-03** | TS-024 | Aprovisionamiento JIT | Como Developer, quiero implementar el aprovisionamiento Just-in-Time (JIT) de usuarios locales usando los claims del JWT de Supabase, para asegurar consistencia de datos. | 3 | Completado |
| 20 | **EPIC-04** | US08 | Carga de CV (PDF/DOCX) | Como usuario, quiero subir mi currículum PDF o DOCX con un área Drag & Drop, para iniciar el análisis. | 3 | Completado |
| 21 | **EPIC-04** | TS-020 | Resiliencia de Archivos | Como Developer, quiero manejar validaciones y archivos corruptos en formatos PDF y DOCX. | 3 | Completado |
| 22 | **EPIC-04** | US10 | Dashboard Diagnóstico | Como usuario, quiero visualizar en un panel Bento-Grid mis fortalezas, brechas y afinidad. | 5 | Completado |
| 23 | **EPIC-04** | US12 | Plan de Acción | Como usuario, quiero obtener un plan de acción para cerrar brechas priorizado determinísticamente ($Prioridad = Peso \times Frecuencia$). | 5 | Completado |
| 24 | **EPIC-04** | US16 | Edición Manual de Habilidades | Como usuario, quiero modificar mis habilidades y datos en caliente para recalcular mi perfil instantáneamente. | 3 | Completado |
| 25 | **EPIC-04** | US19 | Historial de CVs | Como usuario, quiero ver el historial de mis CVs subidos, borrar o re-analizar currículums antiguos. | 3 | Completado |
| 26 | **EPIC-04** | US17 | Vista de Topología del Mercado | Como usuario, quiero interactuar con un grafo de conocimiento de habilidades en 2D/3D (skills-graph). | 5 | Completado |
| 27 | **EPIC-04** | US18 | Comparación por Especialidad | Como usuario, quiero evaluar mi perfil contra otros stacks o clústeres del mercado voluntariamente. | 3 | Completado |
| 28 | **EPIC-05** | TS-014 | Pipeline CI/CD | Como Developer, quiero un pipeline de integración (GitHub Actions), para validar código automáticamente. | 3 | Completado |
| 29 | **EPIC-05** | TS-008 | Auditoría y Logs | Como Developer, quiero un servicio de logging estructurado, para registrar accesos y fallos. | 3 | Completado |
| 30 | **EPIC-05** | TS-006 | Contenerización Docker | Como Developer, quiero configurar Dockerfiles optimizados, para asegurar portabilidad y despliegue. | 5 | Completado |

---

## 🚫 Historias Clasificadas como Post-MVP (Alcance Futuro)

Para asegurar la viabilidad del MVP, las siguientes historias se clasifican en el backlog Post-MVP:
*   **US20 (Exportación de CV ATS):** Exportador dinámico de currículums optimizados con IA basados en brechas cerradas.
*   **TS-011 / TS-013 (Orquestación en la nube):** Procesamiento batch automatizado y periódico del scraper laboral en la nube (se ejecuta de forma manual local en el MVP).

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
- [📄 Documento de Requerimientos de Producto (PRD)](PRD.md)
- [🏃 Sprint Backlog](SPRINT_BACKLOG.md)
