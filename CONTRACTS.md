# 🤝 Contratos de Interfaz - Devalign

Este documento especifica los contratos de API HTTP (REST) entre la aplicación Web frontend (Next.js) y la API del backend (FastAPI) para el MVP.

## 🔐 Autenticación

Todas las peticiones a endpoints protegidos deben incluir el token de acceso JWT provisto por Supabase Auth en las cabeceras HTTP.

```http
Authorization: Bearer <JWT_TOKEN>
```

---

## 🔗 Endpoints de la API

### 1. `GET /api/v1/users/me`
Aprovisionamiento Just-In-Time (JIT) de usuarios. Se invoca tras el inicio de sesión exitoso en el cliente para asegurar la sincronización del perfil del usuario en la base de datos de negocio.

* **Método:** `GET`
* **Cabeceras obligatorias:** `Authorization: Bearer <JWT>`
* **Código de respuesta exitosa:** `200 OK` (Usuario verificado/creado)
* **Respuesta (JSON):**
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "user@example.com",
  "name": "John Doe",
  "created_at": "2026-06-20T17:00:00Z",
  "is_active": true
}
```

---

### 2. `POST /api/v1/users/me/cv`
Carga el archivo del currículum (CV) en formato binario, guarda el documento original en Supabase Storage, extrae el texto e inicia la normalización de habilidades.

* **Método:** `POST`
* **Tipo de Contenido:** `multipart/form-data`
* **Cabeceras obligatorias:** `Authorization: Bearer <JWT>`
* **Cuerpo de la petición:**
  - `file`: Archivo binario (PDF o DOCX, máx. 5MB)
* **Códigos de respuesta:**
  - `202 Accepted`: Archivo subido y procesamiento iniciado con éxito.
  - `400 Bad Request`: Formato de archivo no válido o excedido en tamaño.
* **Respuesta (JSON):**
```json
{
  "message": "El documento CV ha sido cargado exitosamente. Procesamiento de análisis en curso.",
  "document_id": "8a728b9c-293e-4b2b-a19f-092db19463b2",
  "status": "processing"
}
```

---

### 3. `GET /api/v1/profile/me`
Obtiene el JSON de Diagnóstico del usuario autenticado, que mapea el perfil profesional inferido contra el mercado actual.

* **Método:** `GET`
* **Cabeceras obligatorias:** `Authorization: Bearer <JWT>`
* **Códigos de respuesta:**
  - `200 OK`: Diagnóstico retornado con éxito.
  - `404 Not Found`: El usuario no cuenta con un CV procesado o un perfil configurado.
* **Respuesta (JSON):**
```json
{
  "profile": {
    "title": "Backend Software Engineer",
    "experience_level": "Senior",
    "updated_at": "2026-06-20T17:05:00Z"
  },
  "alignment": {
    "matched_cluster": "Python Backend & Cloud Architecture",
    "similarity_score": 0.825,
    "matching_skills": [
      { "name": "Python", "category": "Languages", "proficiency": "Expert" },
      { "name": "FastAPI", "category": "Frameworks", "proficiency": "Intermediate" },
      { "name": "PostgreSQL", "category": "Databases", "proficiency": "Intermediate" }
    ],
    "missing_skills": [
      { "name": "Docker", "category": "DevOps", "weight": 0.9, "frequency_in_cluster": 0.88 },
      { "name": "Kubernetes", "category": "DevOps", "weight": 0.7, "frequency_in_cluster": 0.65 },
      { "name": "Redis", "category": "Databases", "weight": 0.5, "frequency_in_cluster": 0.55 }
    ]
  },
  "action_recommendations": [
    {
      "skill_name": "Docker",
      "priority_score": 0.792,
      "gap_priority": "High",
      "recommended_action": "Aprender contenerización básica de servicios de Python usando Docker y Docker-compose."
    },
    {
      "skill_name": "Kubernetes",
      "priority_score": 0.455,
      "gap_priority": "Medium",
      "recommended_action": "Familiarizarse con conceptos clave como Pods, Deployments y Services."
    }
  ]
}
```

> **Nota de Implementación del MVP:**
> En el MVP, las recomendaciones de acción se generan de forma determinista basándose en la prioridad de las brechas técnicas ($Prioridad = Peso \times Frecuencia$). No requiere llamadas síncronas/asíncronas generativas adicionales en tiempo de ejecución.

---

### 4. `PUT /api/v1/profile/skills`
Permite al usuario agregar, eliminar o modificar manualmente sus habilidades declaradas en el panel para recalcular la alineación profesional sin tener que subir su CV de nuevo.

* **Método:** `PUT`
* **Cabeceras obligatorias:** `Authorization: Bearer <JWT>`
* **Cuerpo de la petición (JSON):**
```json
{
  "skills": [
    { "name": "Python", "action": "add" },
    { "name": "Docker", "action": "add" },
    { "name": "Kubernetes", "action": "remove" }
  ]
}
```
* **Códigos de respuesta:**
  - `200 OK`: Perfil actualizado exitosamente.
* **Respuesta (JSON):**
```json
{
  "message": "Habilidades del perfil actualizadas correctamente.",
  "updated_skills_count": 2
}
```

---

### 5. `GET /api/v1/profile/skills-graph`
Retorna una representación en formato JSON de grafo del mapa de conocimiento del usuario, mostrando las conexiones entre sus habilidades actuales y las brechas del clúster sugerido.

* **Método:** `GET`
* **Cabeceras obligatorias:** `Authorization: Bearer <JWT>`
* **Códigos de respuesta:**
  - `200 OK`: Grafo retornado con éxito.
* **Respuesta (JSON):**
```json
{
  "nodes": [
    { "id": "1", "label": "Python", "type": "current", "category": "Languages" },
    { "id": "2", "label": "FastAPI", "type": "current", "category": "Frameworks" },
    { "id": "3", "label": "Docker", "type": "missing", "category": "DevOps" }
  ],
  "edges": [
    { "source": "1", "target": "2", "relation": "framework_of" },
    { "source": "2", "target": "3", "relation": "deploys_with" }
  ]
}
```

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
