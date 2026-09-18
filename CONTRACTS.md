# 🤝 Contratos de Interfaz

Este documento especifica los contratos formales de la API HTTP REST entre la aplicación Web frontend (`devalign-web`) y la API backend (`devalign-api`), reflejando con exactitud los 21 endpoints implementados y sus estructuras de datos (DTOs).

---

## Autenticación y Seguridad

Todas las peticiones a endpoints protegidos deben incluir el token de acceso JWT provisto por **Supabase Auth** en la cabecera HTTP:

```http
Authorization: Bearer <JWT_TOKEN>
```

- **Prefijo base de la API:** `/api/v1`
- **Aprovisionamiento JIT (Just-In-Time):** Si un usuario autenticado en Supabase realiza su primera llamada a `/api/v1/me`, la API sincroniza automáticamente el registro en la tabla pública `users`.
- **CORS:** Configurado para admitir orígenes locales y expresiones regulares (`CORS_ORIGIN_REGEX`) para previsualizaciones dinámicas en Vercel.

---

## Endpoints de Usuario y CV (`/api/v1/me`)

### 1. `GET /api/v1/me`
**Descripción:** Retorna el perfil completo del desarrollador autenticado con su diagnóstico consolidado, afinidades de clúster, afinidades de dominio y brechas de habilidades. Si no ha analizado ningún CV, retorna un borrador inicial.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Respuesta `200 OK`:**
```json
{
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "email": "dev@ejemplo.com",
  "full_name": "Dev User",
  "avatar_url": "https://example.com/avatar.jpg",
  "created_at": "2026-09-18T10:00:00Z",
  "seniority": "Senior",
  "primary_specialty": "Backend Cloud & Distributed Systems",
  "alignment_score": 0.84,
  "ict_score": 7.8,
  "is_diagnosed": true,
  "detected_skills": [
    {
      "name": "Python",
      "skill_type": "tech",
      "years_of_experience": 5,
      "self_taught": false,
      "personal_projects": true,
      "has_certification": true,
      "ict_score": 8.0,
      "inferred_from": [],
      "core_domains": ["Backend"],
      "domain_tags": ["Python", "FastAPI"]
    }
  ],
  "skill_gaps": [
    {
      "name": "Kubernetes",
      "skill_type": "tech",
      "market_importance": "critical",
      "inferred_from": [],
      "core_domains": ["DevOps & Cloud"],
      "domain_tags": ["Containers", "Orchestration"]
    }
  ],
  "secondary_affinities": [
    {
      "cluster_id": "b3b728b9-1111-2222-3333-444455556666",
      "cluster_name": "Fullstack React & Node",
      "affinity_score": 0.52,
      "is_primary": false,
      "ai_insight": "Cuentas con bases sólidas en backend. Aprender React incrementará tu afinidad a 80%.",
      "market_insights": {
        "average_salary_usd": 3200,
        "total_demand": 410,
        "market_share_percentage": 28.5
      },
      "compatible_roles": [
        { "title": "Fullstack Developer", "match": "Media", "frequency": 42 }
      ],
      "detected_skills": [],
      "skill_gaps": []
    }
  ],
  "domain_affinities": [
    { "domain": "Backend", "affinity_score": 0.88 },
    { "domain": "DevOps & Cloud", "affinity_score": 0.61 }
  ],
  "current_job_role": "Senior Backend Developer",
  "years_experience": 6,
  "professional_summary": "Especialista en arquitecturas distribuidas con FastAPI y cloud computing.",
  "preferred_modality": "Remoto",
  "location": "Lima, Perú",
  "availability": "Inmediata",
  "work_experience": [
    {
      "company": "Fintech Solutions",
      "role": "Backend Engineer",
      "description": "Diseño de microservicios transaccionales de alta concurrencia.",
      "start_date": "2022-01-01",
      "end_date": null,
      "current": true
    }
  ],
  "education": [],
  "certifications": [],
  "message": "Perfil cargado con éxito"
}
```

---

### 2. `PATCH /api/v1/me`
**Descripción:** Actualiza los campos manuales del perfil (datos personales, resumen profesional, modalidad, experiencia, educación y certificaciones).

- **Método:** `PATCH`
- **Autenticación:** Obligatoria
- **Request Body (`ProfileUpdateDTO`):**
```json
{
  "full_name": "Dev User Actualizado",
  "current_job_role": "Lead Architect",
  "professional_summary": "Arquitecto de software con más de 7 años de experiencia.",
  "years_experience": 7,
  "preferred_modality": "Remoto",
  "location": "Remoto LATAM",
  "availability": "2 semanas",
  "work_experience": [],
  "education": [],
  "certifications": []
}
```
- **Respuesta `200 OK`:** Objeto `UserProfileDTO` actualizado.

---

### 3. `DELETE /api/v1/me`
**Descripción:** Elimina permanentemente la cuenta del usuario, purgando sus archivos de Supabase Storage, registros relacionales en PostgreSQL y su identidad en Supabase Auth.

- **Método:** `DELETE`
- **Autenticación:** Obligatoria
- **Respuesta `204 No Content`**

---

### 4. `POST /api/v1/me/reset`
**Descripción:** Restablece los datos de diagnóstico y documentos CV del usuario sin eliminar su cuenta de autenticación.

- **Método:** `POST`
- **Autenticación:** Obligatoria
- **Respuesta `204 No Content`**

---

### 5. `POST /api/v1/me/cv`
**Descripción:** Carga un archivo CV en formato PDF o DOCX (máximo 5MB), lo persiste en Supabase Storage e inicia la Fase 1 de extracción estructurada en segundo plano.

- **Método:** `POST`
- **Autenticación:** Obligatoria
- **Content-Type:** `multipart/form-data`
- **Request Body:** `file` (archivo binario multipart)
- **Respuesta `201 Created` (`CVUploadResultDTO`):**
```json
{
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "storage_path": "cvs/3fa85f64-.../8a728b9c.pdf",
  "original_filename": "cv_2026.pdf",
  "size_bytes": 1048576,
  "download_url": "https://supabase.co/storage/v1/object/sign/cvs/...",
  "uploaded_at": "2026-09-18T10:00:00Z"
}
```

---

### 6. `GET /api/v1/me/cv/status`
**Descripción:** Obtiene el estado de procesamiento del último CV cargado por el usuario.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Respuesta `200 OK` (`CVStatusDTO`):**
```json
{
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "status": "skills_detected",
  "uploaded_at": "2026-09-18T10:00:00Z",
  "error_message": null
}
```
*Estados posibles del ciclo de vida:* `none`, `processing`, `skills_detected`, `completed`, `failed`.

---

### 7. `GET /api/v1/me/cvs`
**Descripción:** Lista el historial completo de documentos CV subidos por el usuario.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Respuesta `200 OK` (`CVListDTO`):**
```json
{
  "cvs": [
    {
      "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
      "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "storage_path": "cvs/3fa85f64-.../8a728b9c.pdf",
      "original_filename": "cv_backend_2026.pdf",
      "size_bytes": 1048576,
      "download_url": "https://supabase.co/storage/v1/...",
      "uploaded_at": "2026-09-18T10:00:00Z"
    }
  ]
}
```

---

### 8. `GET /api/v1/me/cvs/{cv_id}/status`
**Descripción:** Obtiene el estado detallado de un CV específico. Cuando el estado es `skills_detected`, incluye la lista de habilidades extraídas (`extracted_skills`) con sus evidencias para validación humana en la UI.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cv_id` (UUID)
- **Respuesta `200 OK` (`CVStatusDTO`):**
```json
{
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "status": "skills_detected",
  "uploaded_at": "2026-09-18T10:00:00Z",
  "error_message": null,
  "extracted_skills": [
    {
      "name": "Python",
      "skill_type": "technical",
      "years_of_experience": 5,
      "self_taught": false,
      "personal_projects": true,
      "has_certification": true,
      "is_custom": false,
      "suggested_canonical": null,
      "ict_score": 8.0
    },
    {
      "name": "FastAPI",
      "skill_type": "technical",
      "years_of_experience": 3,
      "self_taught": true,
      "personal_projects": true,
      "has_certification": false,
      "is_custom": false,
      "suggested_canonical": null,
      "ict_score": 6.0
    }
  ]
}
```

---

### 9. `POST /api/v1/me/cvs/{cv_id}/reanalyze`
**Descripción:** Descarga el archivo original desde Supabase Storage y vuelve a disparar la Fase 1 de extracción estructurada.

- **Método:** `POST`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cv_id` (UUID)
- **Respuesta `200 OK` (`CVUploadResultDTO`)**

---

### 10. `POST /api/v1/me/cv/{cv_id}/finalize`
**Descripción:** Dispara la Fase 2 del análisis con las habilidades validadas por el usuario. Ejecuta la normalización contra el catálogo Lightcast, inferencia ascendente, cálculo de afinidad Weighted Jaccard y consolidación del perfil de forma asíncrona.

- **Método:** `POST`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cv_id` (UUID)
- **Request Body (`FinalizeRequestDTO`, opcional):**
```json
{
  "skills": [
    {
      "name": "Python",
      "skill_type": "technical",
      "years_of_experience": 5,
      "self_taught": false,
      "personal_projects": true,
      "has_certification": true
    },
    {
      "name": "Docker",
      "skill_type": "technical",
      "years_of_experience": 2,
      "self_taught": true,
      "personal_projects": true,
      "has_certification": false
    }
  ]
}
```
- **Respuesta `202 Accepted` (`FinalizeResponseDTO`):**
```json
{
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "status": "processing",
  "message": "Diagnóstico en proceso..."
}
```

---

### 11. `DELETE /api/v1/me/cvs/{cv_id}`
**Descripción:** Elimina un documento CV del historial y su archivo asociado en Supabase Storage.

- **Método:** `DELETE`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cv_id` (UUID)
- **Respuesta `204 No Content`**

---

### 12. `PUT /api/v1/me/skills`
**Descripción:** Sobrescribe manualmente las habilidades del perfil activo y recalcula en caliente el diagnóstico, las afinidades y las brechas.

- **Método:** `PUT`
- **Autenticación:** Obligatoria
- **Request Body (`SkillsUpdateDTO`):**
```json
{
  "skills": [
    { "name": "Python", "skill_type": "tech" },
    { "name": "PostgreSQL", "skill_type": "tech" },
    { "name": "Docker", "skill_type": "tech" }
  ]
}
```
- **Respuesta `200 OK`:** Objeto `UserProfileDTO` recalculado.

---

### 13. `POST /api/v1/me/affinities/{cluster_name}`
**Descripción:** Evalúa el perfil actual del desarrollador contra un clúster específico del mercado laboral, registrando los resultados en sus afinidades secundarias.

- **Método:** `POST`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cluster_name` (string)
- **Respuesta `200 OK`:** Objeto `UserProfileDTO` con la nueva afinidad evaluada.

---

### 14. `GET /api/v1/me/diagnostics/{cluster_name}`
**Descripción:** Obtiene el diagnóstico consolidado y las estadísticas de mercado para un clúster específico.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Path Parameters:** `cluster_name` (string)
- **Respuesta `200 OK` (`DiagnosticDetailDTO`):**
```json
{
  "cluster_name": "Cloud Backend Python",
  "affinity_score": 0.82,
  "seniority": "Senior",
  "consolidated_skills": [
    { "name": "Python", "skill_type": "tech", "ict_score": 8.0 }
  ],
  "gap_skills": [
    { "name": "Terraform", "skill_type": "tech", "market_importance": "critical" }
  ],
  "emerging_skills": [],
  "market_insights": {
    "total_demand": 380,
    "average_salary_usd": 3500,
    "growth_percentage": 14.2
  },
  "compatible_roles": [
    { "title": "Senior Cloud Backend Engineer", "match": "Alta", "frequency": 55 }
  ]
}
```

---

### 15. `GET /api/v1/me/skills/search`
**Descripción:** Búsqueda autocompletada de habilidades canónicas y aliases en el catálogo de gobernanza.

- **Método:** `GET`
- **Autenticación:** Obligatoria
- **Query Parameters:**
  - `q` (string, obligatorio): Término de búsqueda.
  - `limit` (int, opcional, default: 20): Cantidad máxima de resultados.
- **Respuesta `200 OK` (`list[SkillSearchResultDTO]`):**
```json
[
  {
    "id": "e4b1029c-5555-4444-3333-222211110000",
    "name": "FastAPI",
    "skill_type": "tech",
    "status": "canonical",
    "standard_name": "Lightcast",
    "standard_type": "Specialized Skill",
    "category_name": "Information Technology",
    "subcategory_name": "Software Development",
    "domain_tags": ["Python", "API", "Web Framework"],
    "core_domains": ["Backend"],
    "matched_alias": null
  }
]
```

---

## Endpoints de Inteligencia de Mercado (`/api/v1/market`)

### 16. `GET /api/v1/market/skills/search`
**Descripción:** Endpoint público de autocompletado de habilidades para exploradores de mercado y formularios públicos.

- **Método:** `GET`
- **Autenticación:** Opcional
- **Query Parameters:** `q` (string), `limit` (int)
- **Respuesta `200 OK`:** `list[SkillSearchResultDTO]` (idéntico al endpoint `/me/skills/search`).

---

### 17. `GET /api/v1/market/clusters`
**Descripción:** Retorna todas las especialidades tecnológicas descubiertas mediante el pipeline de clustering en `devalign-ml`.

- **Método:** `GET`
- **Autenticación:** Opcional
- **Respuesta `200 OK` (`list[ClusterDTO]`):**
```json
[
  {
    "cluster_id": "b3b728b9-1111-2222-3333-444455556666",
    "name": "Backend Python & Cloud Distributed",
    "description": "Dominant skills: Python, Docker, FastAPI, AWS, PostgreSQL. Size: 520 offers.",
    "job_offer_count": 520,
    "top_skills": [
      { "name": "Python", "importance_score": 2.7, "frequency": 0.90 },
      { "name": "Docker", "importance_score": 2.4, "frequency": 0.80 }
    ],
    "compatible_roles": [
      { "title": "Backend Developer", "match": "Alta", "frequency": 120 }
    ],
    "market_insights": {
      "growth_percentage": 18.5,
      "market_share_percentage": 24.2,
      "salary_differential": "N/A"
    }
  }
]
```

---

### 18. `GET /api/v1/market/skills-graph`
**Descripción:** Retorna el grafo de conocimiento relacional completo de habilidades (nodos y aristas). Si el usuario está autenticado, superpone el estado de sus habilidades adquiridas y brechas para renderizado interactivo en 2D/3D.

- **Método:** `GET`
- **Autenticación:** Opcional
- **Query Parameters:** `cluster` (string, opcional)
- **Respuesta `200 OK` (`GraphResponseDTO`):**
```json
{
  "nodes": [
    {
      "id": "FastAPI",
      "name": "FastAPI",
      "nature": "tech",
      "group": "Backend",
      "weight": 2.0,
      "status": "user_acquired",
      "domain_tags": ["Python", "Web"]
    },
    {
      "id": "Python",
      "name": "Python",
      "nature": "tech",
      "group": "Backend",
      "weight": 3.0,
      "status": "user_acquired",
      "domain_tags": ["Programming Languages"]
    }
  ],
  "links": [
    {
      "source": "FastAPI",
      "target": "Python",
      "relation_type": "requires",
      "weight": 1.0
    }
  ]
}
```

---

## Endpoints de Administración y Operaciones

### 19. `POST /api/v1/admin/skills/normalize`
**Descripción:** Dispara el proceso de normalización masiva de habilidades crudas en las ofertas laborales (`job_offers.raw_hard_skills`) contra el catálogo canónico.

- **Método:** `POST`
- **Autenticación:** Administrativa
- **Respuesta `200 OK`:**
```json
{
  "processed_offers": 1420,
  "normalized_skills_count": 8950,
  "new_aliases_registered": 124,
  "status": "success"
}
```

---

### 20. `GET /api/v1/scraper/status`
**Descripción:** Endpoint informativo para verificar el estado de los conjuntos de datos recopilados por `devalign-scraping`.

- **Método:** `GET`
- **Respuesta `200 OK`:**
```json
{
  "status": "stub",
  "job_offer_count": 0,
  "message": "Scraper module is decoupled and populates PostgreSQL directly via Supabase exporter."
}
```

---

### 21. `GET /health`
**Descripción:** Verificación de salud del servicio y entorno de ejecución.

- **Método:** `GET`
- **Respuesta `200 OK`:**
```json
{
  "status": "healthy",
  "version": "0.1.0",
  "env": "production"
}
```

---

## Códigos de Estado y Errores Estándar

| Código HTTP | Significado | Causa común |
|---|---|---|
| `200 OK` | Operación exitosa | Lectura o actualización síncrona completada. |
| `201 Created` | Recurso creado | Documento CV cargado en Supabase Storage y registrado en DB. |
| `202 Accepted` | Petición aceptada | Fase 2 de diagnóstico delegada a background task. |
| `204 No Content` | Sin contenido | Eliminación exitosa o reinicio de cuenta. |
| `400 Bad Request` | Petición inválida | Archivo no soportado o formato de datos corrupto. |
| `401 Unauthorized` | No autorizado | Token JWT ausente, expirado o inválido. |
| `404 Not Found` | No encontrado | Recurso o diagnóstico no existente para el usuario. |
| `409 Conflict` | Conflicto | Intento de finalizar un CV que aún está en estado `processing`. |
| `429 Too Many Requests` | Rate limit excedido | Límites de peticiones a servicios LLM o de almacenamiento. |
| `500 Internal Server Error` | Error de servidor | Excepción no controlada en pipeline de análisis. |

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
- [📄 Documento de Requerimientos de Producto (PRD)](PRD.md)
- [📋 Product Backlog](PRODUCT_BACKLOG.md)
- [🏃 Sprint Backlog](SPRINT_BACKLOG.md)
