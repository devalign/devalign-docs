# 🤝 Contratos de Interfaz

Este documento especifica los contratos de API HTTP (REST) entre la aplicación Web frontend (Next.js 16) y la API del backend (FastAPI) de Devalign, reflejando con exactitud los endpoints implementados y consumidos.

## Autenticación

Todas las peticiones a endpoints protegidos deben incluir el token de acceso JWT provisto por Supabase Auth en las cabeceras HTTP:

```http
Authorization: Bearer <JWT_TOKEN>
```

El prefijo base de todos los endpoints es `/api/v1`.

---

## Endpoints de Usuario y CV

### 1. `GET /api/v1/me`
Aprovisionamiento Just-In-Time (JIT) del usuario autenticado. Crea el perfil local si es primera vez y retorna el perfil completo con diagnóstico.

- **Método:** `GET`
- **Respuesta:** `200 OK`
- **Content-Type:** `application/json`

```json
{
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "dev@example.com",
  "full_name": "John Doe",
  "avatar_url": "https://example.com/avatar.jpg",
  "created_at": "2026-06-20T17:00:00Z",
  "seniority": "Senior",
  "primary_specialty": "Python Backend & Cloud Architecture",
  "alignment_score": 0.82,
  "detected_skills": [
    { "name": "Python", "skill_type": "tech", "market_importance": null, "inferred_from": [] },
    { "name": "FastAPI", "skill_type": "tech", "market_importance": null, "inferred_from": [] }
  ],
  "skill_gaps": [
    { "name": "Docker", "skill_type": "tech", "market_importance": "critical", "inferred_from": [] }
  ],
  "secondary_affinities": [
    {
      "cluster_id": "b3b728b9-...",
      "cluster_name": "Fullstack Node & React",
      "affinity_score": 0.45,
      "is_primary": false,
      "ai_insight": "Dominas 1 tecnologías clave. Para mejorar, considera aprender React.",
      "market_insights": { "average_salary_pen": 7500, "total_demand": 320 },
      "compatible_roles": [{ "title": "Fullstack Developer", "match": "Media" }],
      "detected_skills": [],
      "skill_gaps": []
    }
  ],
  "domain_affinities": [
    { "domain": "Backend", "affinity_score": 0.78 },
    { "domain": "Database", "affinity_score": 0.65 }
  ],
  "ict_score": 7.5,
  "current_job_role": "Backend Engineer",
  "years_experience": 6,
  "professional_summary": "Desarrollador backend con experiencia en...",
  "preferred_modality": "Remoto",
  "location": "Lima, Perú",
  "availability": "Inmediata",
  "work_experience": [
    {
      "company": "Tech Corp",
      "role": "Backend Developer",
      "description": "Microservicios con FastAPI.",
      "start_date": "2022-01-01",
      "end_date": null,
      "current": true
    }
  ],
  "education": [],
  "certifications": [],
  "message": "Profile generated successfully"
}
```

### 2. `PATCH /api/v1/me`
Actualiza los campos manuales del perfil del desarrollador.

- **Método:** `PATCH`
- **Content-Type:** `application/json`
- **Cuerpo:**
```json
{
  "full_name": "John Doe Editado",
  "current_job_role": "Senior Backend Developer",
  "years_experience": 7,
  "preferred_modality": "Híbrido",
  "location": "Lima, Perú",
  "availability": "1 mes",
  "work_experience": [],
  "education": [],
  "certifications": []
}
```
- **Respuesta:** `200 OK` (objeto `UserProfileDTO` completo)

### 3. `DELETE /api/v1/me`
Elimina permanentemente la cuenta del usuario y todos sus datos asociados.

- **Método:** `DELETE`
- **Respuesta:** `204 No Content`

### 4. `POST /api/v1/me/cv`
Carga un archivo de currículum (PDF o DOCX, máx. 5MB), lo guarda en Supabase Storage e inicia la extracción asíncrona.

- **Método:** `POST`
- **Content-Type:** `multipart/form-data`
- **Cuerpo:** `file` (archivo binario)
- **Respuesta:** `201 Created`
```json
{
  "cv_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "storage_path": "cvs/3fa85f64-.../8a728b9c.pdf",
  "original_filename": "mi_cv_2026.pdf",
  "size_bytes": 1048576,
  "download_url": "https://supabase.co/storage/v1/object/sign/cvs/...",
  "message": "CV uploaded and analysis scheduled in background."
}
```

### 5. `GET /api/v1/me/cv/status`
Obtiene el estado de procesamiento del CV activo del usuario.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
{
  "cv_id": "8a728b9c-...",
  "status": "extracted",
  "message": "CV extraction completed. Ready to finalize."
}
```
Estados posibles: `processing`, `extracted`, `normalizing`, `completed`, `error`.

### 6. `POST /api/v1/me/cv/{cv_id}/finalize`
Finaliza el análisis del CV ejecutando la Fase 2: normalización de skills, inferencia ascendente, cálculo de afinidad y persistencia del diagnóstico.

- **Método:** `POST`
- **Parámetros:** `cv_id` (UUID, path)
- **Cuerpo (opcional):**
```json
{
  "skills": [
    { "name": "Python", "skill_type": "tech" },
    { "name": "FastAPI", "skill_type": "tech" }
  ]
}
```
- **Respuesta:** `200 OK` (objeto `UserProfileDTO` completo)

### 7. `GET /api/v1/me/cvs`
Obtiene el historial de documentos CV del usuario.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
{
  "user_id": "3fa85f64-...",
  "cvs": [
    {
      "cv_id": "8a728b9c-...",
      "user_id": "3fa85f64-...",
      "storage_path": "cvs/...",
      "original_filename": "mi_cv_2026.pdf",
      "size_bytes": 1048576,
      "status": "extracted",
      "download_url": "https://...",
      "uploaded_at": "2026-06-20T17:01:00Z"
    }
  ],
  "total": 1
}
```

### 8. `GET /api/v1/me/cvs/{cv_id}/status`
Obtiene el estado de procesamiento de un CV específico.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
{
  "cv_id": "8a728b9c-...",
  "status": "processing",
  "message": "CV is being processed..."
}
```

### 9. `POST /api/v1/me/cvs/{cv_id}/reanalyze`
Dispara un nuevo análisis sobre un CV existente.

- **Método:** `POST`
- **Respuesta:** `200 OK`

### 10. `DELETE /api/v1/me/cvs/{cv_id}`
Elimina un CV del historial y del almacenamiento físico.

- **Método:** `DELETE`
- **Respuesta:** `204 No Content`

### 11. `POST /api/v1/me/reset`
Elimina permanentemente todos los CVs, diagnósticos y datos de perfil del usuario. Mantiene la cuenta activa.

- **Método:** `POST`
- **Respuesta:** `204 No Content`

---

## Endpoints de Perfil e Inferencia

### 12. `PUT /api/v1/me/skills`
Sobrescribe las habilidades del perfil y recalcula afinidades y brechas dinámicamente.

- **Método:** `PUT`
- **Content-Type:** `application/json`
- **Cuerpo:**
```json
{
  "skills": [
    { "name": "Python", "skill_type": "tech" },
    { "name": "FastAPI", "skill_type": "tech" },
    { "name": "Docker", "skill_type": "tech" }
  ]
}
```
- **Respuesta:** `200 OK` (objeto `UserProfileDTO` completo)

### 13. `POST /api/v1/me/affinities/{cluster_name}`
Evalúa el perfil del usuario contra un clúster específico y guarda el resultado en afinidades secundarias.

- **Método:** `POST`
- **Parámetros:** `cluster_name` (string, URL-encoded)
- **Respuesta:** `200 OK` (objeto `UserProfileDTO` actualizado)

### 14. `GET /api/v1/me/diagnostics/{cluster_name}`
Obtiene el detalle completo del diagnóstico contra un clúster específico.

- **Método:** `GET`
- **Parámetros:** `cluster_name` (string, URL-encoded)
- **Respuesta:** `200 OK`

---

## Endpoints de Mercado

### 15. `GET /api/v1/market/clusters`
Retorna el catálogo completo de especialidades técnicas del mercado.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
[
  {
    "id": "a1b2c3d4-...",
    "name": "Python Backend & Cloud Architecture",
    "description": "Especialidad enfocada en desarrollo con Python, APIs modernas y nube.",
    "top_skills": ["Python", "FastAPI", "PostgreSQL", "Docker", "AWS"],
    "job_offer_count": 482
  }
]
```

### 16. `GET /api/v1/market/skills-graph`
Retorna la estructura completa del grafo de conocimiento del mercado. Si el usuario está autenticado, detalla el estado de cada nodo.

- **Método:** `GET`
- **Cabeceras (opcional):** `Authorization: Bearer <JWT>`
- **Respuesta:** `200 OK`
```json
{
  "nodes": [
    { "id": "python", "label": "Python", "group": "Languages", "domains": ["backend"], "status": "acquired" },
    { "id": "fastapi", "label": "FastAPI", "group": "Frameworks", "domains": ["backend"], "status": "acquired" },
    { "id": "docker", "label": "Docker", "group": "DevOps", "domains": ["devops"], "status": "gap" }
  ],
  "links": [
    { "source": "fastapi", "target": "python", "value": 1.0, "type": "explicit_relation" },
    { "source": "python", "target": "sql", "value": 1.0, "type": "implicit_domain" }
  ]
}
```

---

## Endpoints de Administración y Sistema

### 17. `POST /api/v1/admin/skills/normalize`
Ejecuta el pipeline de normalización de skills para ofertas de trabajo pendientes.

- **Método:** `POST`
- **Respuesta:** `200 OK`

### 18. `GET /api/v1/scraper/status`
Endpoint stub que retorna el estado del scraper.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
{
  "status": "stub"
}
```

---

## Endpoint Público

### 19. `GET /health`
Health check del servicio.

- **Método:** `GET`
- **Respuesta:** `200 OK`
```json
{
  "status": "healthy",
  "timestamp": "2026-07-03T12:00:00Z"
}
```

---

## Resumen de Endpoints

| # | Método | Path | Descripción |
|---|--------|------|-------------|
| 1 | `GET` | `/health` | Health check |
| 2 | `GET` | `/api/v1/me` | Perfil completo (JIT provisioning) |
| 3 | `PATCH` | `/api/v1/me` | Actualizar perfil |
| 4 | `DELETE` | `/api/v1/me` | Eliminar cuenta |
| 5 | `PUT` | `/api/v1/me/skills` | Actualizar skills manualmente |
| 6 | `POST` | `/api/v1/me/cv` | Subir CV |
| 7 | `GET` | `/api/v1/me/cv/status` | Estado del CV activo |
| 8 | `POST` | `/api/v1/me/cv/{cv_id}/finalize` | Finalizar análisis |
| 9 | `GET` | `/api/v1/me/cvs` | Listar CVs |
| 10 | `GET` | `/api/v1/me/cvs/{cv_id}/status` | Estado de un CV específico |
| 11 | `POST` | `/api/v1/me/cvs/{cv_id}/reanalyze` | Re-analizar CV |
| 12 | `DELETE` | `/api/v1/me/cvs/{cv_id}` | Eliminar CV |
| 13 | `POST` | `/api/v1/me/reset` | Resetear datos de usuario |
| 14 | `POST` | `/api/v1/me/affinities/{cluster_name}` | Evaluar contra clúster |
| 15 | `GET` | `/api/v1/me/diagnostics/{cluster_name}` | Diagnóstico detallado de clúster |
| 16 | `GET` | `/api/v1/market/clusters` | Catálogo de clústeres |
| 17 | `GET` | `/api/v1/market/skills-graph` | Grafo de conocimiento |
| 18 | `POST` | `/api/v1/admin/skills/normalize` | Normalizar skills batch |
| 19 | `GET` | `/api/v1/scraper/status` | Estado del scraper (stub) |

---

## Referencias

- [Arquitectura Técnica](ARCHITECTURE.md)
- [Modelo de Base de Datos](DATABASE.md)
- [Lógica Core e Inferencia](MODEL.md)
- [Roadmap de Producto](ROADMAP.md)
- [Alcance MVP](SCOPE.md)
- [Documento de Requerimientos de Producto (PRD)](PRD.md)
- [Product Backlog](PRODUCT_BACKLOG.md)
- [Sprint Backlog](SPRINT_BACKLOG.md)
