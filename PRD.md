# 📄 Documento de Requerimientos de Producto (PRD) - Devalign

Este documento define los requisitos funcionales, el alcance técnico inicial y las decisiones de producto para Devalign.

---

## 🎯 1. Título Propuesto

**Aplicación web con motor de inferencia basado en Machine Learning para el análisis de competencias técnicas demandadas por el sector IT orientado a desarrolladores del Perú.**

---

## Problemática y Alcance

### Problema
Existe una asimetría de información crítica entre la formación técnica de los desarrolladores y la demanda real del sector IT. El problema no es solo la falta de experiencia, sino la **desalineación técnica**: los desarrolladores enfrentan una sobrecarga de información y carecen de una ruta clara para especializarse en los nichos tecnológicos (stacks) que el mercado realmente demanda y valora.

### Propósito
Desarrollar un sistema inteligente que permita identificar automáticamente la especialidad técnica de un desarrollador, diagnosticar su nivel de alineación con el mercado actual y priorizar el cierre de sus brechas técnicas.

### Alcance
El sistema será capaz de:

- **Inteligencia de Mercado (Offline):** Recolectar y procesar ofertas laborales mediante Web Scraping para descubrir clústeres tecnológicos vivos en el mercado.
- **Profiling Invisible:** Procesar el CV del usuario mediante procesamiento de lenguaje natural (NLP) para identificar su afinidad técnica sin necesidad de formularios extensos.
- **Detección de Brechas:** Identificar las habilidades faltantes (hard y soft) comparando el perfil del usuario contra el clúster de competencias de su especialidad.
- **Plan de Acción por Fases:** Generar un plan estructurado por niveles de prioridad a partir de las brechas detectadas.

---

## 🏗️ Arquitectura de Componentes (Enfoque Software)

El sistema se implementará bajo una arquitectura modular desacoplada basada en servicios utilizando **FastAPI**.

### 3.1 Componente de Adquisición (Scraper)
Responsable de la extracción de datos de portales de empleo de alta relevancia.

**Especificaciones técnicas:**
- **Páginas Destino:** Computrabajo (LinkedIn y GetOnBoard se clasifican como alcance Post-MVP debido a complejidades de captcha y términos de servicio).
- **Volumen de datos:** Procesamiento de una muestra representativa de ofertas de empleo activas en el sector TI de Perú.
- **Patrón de diseño:** Implementación de *Strategy Pattern* para garantizar la escalabilidad y adaptabilidad ante diversos esquemas de DOM.
- **Extracción de entidades:** Identificación de `hard_skills`, `soft_skills` y herramientas de software mediante técnicas de normalización léxica.

---

### 🧠 Componente de Inteligencia (Motor Analítico ML)

#### a) Descubrimiento de Especialidades (Clustering)
Utilización del algoritmo **UMAP** para reducción de dimensionalidad (a 15-d) y **HDBSCAN** para el agrupamiento de variables categóricas (presencia/ausencia de habilidades técnicas). El modelo descubre "Especialidades Técnicas Empíricas" (ej. *Python Backend & Cloud Architecture*), determinando la co-ocurrencia estricta de tecnologías en el mercado real. 

> **Decisión de Diseño:** Se ha descartado por completo K-Means y K-Modes para evitar predefinir arbitrariamente el número de agrupaciones ($K$) y tolerar de forma nativa el ruido en los datos de entrada.

#### b) Extracción NLP y Normalización Taxonómica
Procesamiento del currículum vítae (CV) mediante modelos de lenguaje (LLM) estructurados en formato JSON. El documento se procesa para extraer entidades que luego son normalizadas hacia una taxonomía canónica mediante coincidencia exacta $O(1)$ y búsqueda de embeddings semánticos con **Voyage AI** (similitud de coseno $\ge 0.88$).

#### c) Motor de Diagnóstico de Brechas (Alineación)
Análisis de conjuntos que contrasta las habilidades del usuario contra el clúster objetivo utilizando el algoritmo **Weighted Jaccard Similarity** (con créditos del 30% por coincidencia parcial de dominios tecnológicos). Esto permite identificar brechas exactas a nivel de herramientas, lenguajes y metodologías.

---

## 🔌 Componente de Entrega (API REST)
Expone los resultados al frontend, gestionando el flujo de diagnóstico y visualización de perfiles.

### Validación Taxonómica del Modelo ML
El marco teórico valida el modelo de Machine Learning mediante un proceso de contraste taxonómico:
- Los clústeres de tecnologías empíricas obtenidos mediante el scraping se validan utilizando las taxonomías oficiales de **SFIA 9** y las áreas de conocimiento de **SWEBOK/SWECOM**.

### Componente de Gestión de Identidad y Persistencia (Supabase)
Se implementa una infraestructura de servicios basada en **Supabase** para la gestión de datos y usuarios:
- **Gestión de Identidad (Auth):** Sistema de autenticación centralizado con soporte para **OAuth 2.0 via Google**.
- **Almacenamiento de Documentos (Storage):** Repositorio para la persistencia de los currículums vítae cargados.
- **Capa de Datos:** Base de datos relacional PostgreSQL con la extensión `pgvector` para el almacenamiento de embeddings semánticos de 1024 dimensiones.

---

## 🔄 Flujo de Funcionamiento Sistémico

1. **Ingesta de Datos (Offline):** El Scraper recolecta ofertas laborales y extrae metadatos.
2. **Normalización:** Las herramientas de las ofertas se normalizan a un diccionario canónico de habilidades.
3. **Modelado ML (Batch):** El pipeline de UMAP + HDBSCAN agrupa los vectores de las ofertas para descubrir clústeres y calcular los centroides.
4. **Profiling de Usuario (Online):** El CV del usuario es procesado para extraer y normalizar sus habilidades.
5. **Análisis de Brechas:** Se calcula la alineación (Weighted Jaccard) contra los clústeres para detectar brechas explícitas frente al estándar del mercado y priorizarlas determinísticamente ($Prioridad = Peso \times Frecuencia$).

---

## 💡 Sustento de Innovación

### Propuesta Diferenciadora
A diferencia de plataformas de aprendizaje genéricas, este sistema utiliza **Inteligencia de Mercado** para que el aprendizaje no sea teórico, sino dictado por la demanda real y actual del Sector IT.

### Innovaciones Clave
- **Descubrimiento Dinámico de Roles:** El mercado define qué es un especialista, no un currículo estático.
- **Profiling Invisible:** Diagnóstico basado en el CV real, eliminando sesgos de autopercepción.
- **Especialización Profunda:** Enfoque en nichos de alta demanda para maximizar la empleabilidad.

---

## 🛠️ Tecnologías Utilizadas
- **Backend:** FastAPI (Python).
- **IA/ML:** UMAP + HDBSCAN para clustering, OpenAI/Claude API para extracción estructurada, Voyage AI API para normalización semántica.
- **Persistencia y Auth:** Supabase (PostgreSQL con pgvector, Auth, Storage para CVs).
- **Esquema de BD:** SQLAlchemy + Alembic como SSOT.
- **Frontend:** Next.js 16 (App Router), Tailwind CSS v4, pnpm.

---

## 🔗 Referencias

- [🏗️ Arquitectura Técnica](ARCHITECTURE.md)
- [🤝 Contratos de Interfaz](CONTRACTS.md)
- [🗄️ Modelo de Base de Datos](DATABASE.md)
- [🧠 Lógica Core e Inferencia](MODEL.md)
- [🗺️ Roadmap de Producto](ROADMAP.md)
- [🎯 Alcance MVP](SCOPE.md)
- [📋 Product Backlog](PRODUCT_BACKLOG.md)
- [🏃 Sprint Backlog](SPRINT_BACKLOG.md)
