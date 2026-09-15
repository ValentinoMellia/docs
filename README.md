# Documentación Técnica y Arquitectónica — Mercado (Tema 09)
## Plataforma Gamificada Distribuida · Aula Quest (TUP)

Este repositorio centraliza las especificaciones técnicas, protocolos de integración inter-microservicios, propuestas arquitectónicas y contratos de eventos del módulo de **Mercado (Tema 09)**.

---

## 🗂️ Estructura del Repositorio

```
docs/
├── Comunicacion/                             # Contratos y protocolos de comunicación inter-servicios
│   ├── Grupo-01-Identidad-y-Gateway/        # Auth perimetral, JWT y propagación RBAC
│   ├── Grupo-02-Cursos-y-Matricula/         # Validación síncrona y ciclo de vida de cohorte
│   ├── Grupo-03-Motor-de-Desafios/          # HUD de equipamiento y consumo de desafíos resueltos
│   ├── Grupo-08-Banco/                      # Transacciones de compra, HOLDs de subastas y eventos Kafka
│   └── Grupo-12-Backoffice/                 # Parámetros económicos y analítica de ventas
│
├── Propuestas/                              # Propuestas arquitectónicas presentadas a otros equipos
│   └── Grupo-03-Motor-de-Desafios/          # Desacople total con patrón espía / observador
│
├── Mercado/                                 # Especificaciones internas del módulo Mercado
│   ├── Subastas/                            # Arquitectura, resiliencia y contratos de subastas (E-07)
│   ├── Catalogos/                           # Catálogo de consumibles y balance pedagógico
│   └── Arquitectura-General/                # Flujos globales y mapa de integración de microservicios
│
├── Workflow/                                # Guía de Git Workflow, política de ramas y convención de commits
│   ├── README.md                            # Documento normativo y reglas de Pull Request
│   └── diagrama-git-workflow.png            # Diagrama visual de ciclo de vida de ramas
│
└── [Documentos Iniciales]                   # Kickoff, spikes y presentaciones previas
```

---

## 📡 1. Comunicación e Integraciones Inter-Microservicios

| Grupo / Microservicio | Documento | Formato / Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **Grupo 01 — Identidad y Gateway** | [flujo-comunicacion-usuarios.md](./Comunicacion/Grupo-01-Identidad-y-Gateway/flujo-comunicacion-usuarios.md) | Markdown | Seguridad perimetral, validación de JWT e inyección de cabeceras seguras (`X-User-Id`, `X-Roles`). |
| **Grupo 02 — Cursos y Matrícula** | [flujo-comunicacion-cursos.md](./Comunicacion/Grupo-02-Cursos-y-Matricula/flujo-comunicacion-cursos.md) | Markdown | Validación síncrona de matrícula (`GET /enrollment-status`) y suscripción asíncrona a `cursos.ciclo-vida`. |
| **Grupo 03 — Motor de Desafíos** | [flujo-comunicacion-motor-desafio.md](./Comunicacion/Grupo-03-Motor-de-Desafios/flujo-comunicacion-motor-desafio.md) | Markdown | Consulta síncrona de equipamiento en IDE web y suscripción a hechos en `desafios.resultados`. |
| **Grupo 08 — Banco** | [flujo-comunicacion-banco.md](./Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md) | Markdown | Coreografía Kafka con envoltura estándar, modelo HOLD/Saga compensatoria y streaming SSE. |
| **Grupo 08 — Banco** | [Banco-T08_Mercado-T09_Documento-de-integracion.docx](./Comunicacion/Grupo-08-Banco/Banco-T08_Mercado-T09_Documento-de-integracion.docx) | Word (Docx) | Documento formal acordado entre los equipos de Banco y Mercado. |
| **Grupo 08 — Banco** | [flujo-compra-corregido.html](./Comunicacion/Grupo-08-Banco/flujo-compra-corregido.html) | HTML Interactivo | Diagrama visual del flujo de compra y reserva de saldo. |
| **Grupo 08 — Banco** | [flujo-subasta.html](./Comunicacion/Grupo-08-Banco/flujo-subasta.html) | HTML Interactivo | Diagrama visual de pujas y retención de fondos. |
| **Grupo 12 — Backoffice** | [flujo-comunicacion-backoffice.md](./Comunicacion/Grupo-12-Backoffice/flujo-comunicacion-backoffice.md) | Markdown | Consumo asíncrono de `backoffice.parametros` (precios, tiers, vidas) y analítica de ventas. |

---

## 💡 2. Propuestas Arquitectónicas para Otros Equipos

| Destinatario | Documento | Formato / Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **Grupo 03 — Motor de Desafíos** | [propuesta-desacople-motor-desafios.html](./Propuestas/Grupo-03-Motor-de-Desafios/propuesta-desacople-motor-desafios.html) | HTML Interactivo | Análisis y propuesta de desacople arquitectónico del motor de desafíos. |
| **Grupo 03 — Motor de Desafíos** | [propuesta-solucion-espias-desacople.html](./Propuestas/Grupo-03-Motor-de-Desafios/propuesta-solucion-espias-desacople.html) | HTML Interactivo | Solución técnica detallada basada en el patrón "Espía" / Observador. |
| **Grupo 03 — Motor de Desafíos** | [Diagrama SVG](./Propuestas/Grupo-03-Motor-de-Desafios/propuesta-solucion-espias-desacople.svg) / [PNG](./Propuestas/Grupo-03-Motor-de-Desafios/propuesta-solucion-espias-desacople.png) | Gráfico Vectorial / Raster | Esquema visual del patrón espía. |

---

## 🏛️ 3. Módulo Interno de Mercado (Tema 09)

### 3.1 Subastas (Épica E-07)
Directorio completo: [`Mercado/Subastas/`](./Mercado/Subastas/)
* [**README de Subastas**](./Mercado/Subastas/README.md): Resumen e índice del subsistema.
* [**Documento Maestro de Arquitectura (HTML)**](./Mercado/Subastas/documento-arquitectura-subastas.html): Documento interactivo con navegación por secciones, opciones de arquitectura y visor Archify embebido.
* [**01 · Análisis de Opciones de Arquitectura**](./Mercado/Subastas/01-analisis-opciones-arquitectura.md): Evaluación de alternativas (Outbox/Saga, Redis, State Machine) y trade-offs.
* [**02 · Matriz de Fallos, Errores y Resiliencia**](./Mercado/Subastas/02-matriz-fallos-resiliencia-y-soluciones.md): Prevención de desincronizaciones de TTL, concurrencia de pujas y mitigación de errores letales.
* [**03 · Contratos de Eventos e Idempotencia**](./Mercado/Subastas/03-contratos-eventos-e-idempotencia.md): Envoltura oficial de eventos Kafka, claves de deduplicación y estrategias de idempotencia.
* [**Flujo Interactivo Archify (HTML)**](./Mercado/Subastas/flujo-subasta-archify.html) & [Especificación JSON](./Mercado/Subastas/flujo-subasta-archify.json): Diagrama de secuencia navegable por vistas guiadas.

### 3.2 Catálogos y Mecánicas de Juego
Directorio: [`Mercado/Catalogos/`](./Mercado/Catalogos/)
* [**Catálogo de Consumibles (HTML)**](./Mercado/Catalogos/catalogo-items-consumibles.html): Catálogo interactivo con catálogo de ítems, tiers de precios y reglas de consumo.
* [**Evaluación de Ítems (HTML)**](./Mercado/Catalogos/evaluacion-items-1.html): Análisis pedagógico y balance de impacto en la experiencia del alumno.

### 3.3 Arquitectura General
Directorio: [`Mercado/Arquitectura-General/`](./Mercado/Arquitectura-General/)
* [**Diagrama de Flujo Inter-Microservicios (HTML)**](./Mercado/Arquitectura-General/diagrama-flujo-microservicios.html): Mapa general de microservicios y posición de entrada de Mercado.
* [**Flujos de Integración de Mercado (HTML)**](./Mercado/Arquitectura-General/flujos-integracion-mercado.html): Dashboard integral de integración de Mercado con el ecosistema Aula Quest.

---

## 🛠️ 4. Metodología de Desarrollo y Workflow de Git

Directorio: [`Workflow/`](./Workflow/)
* [**Guía Oficial de Git Workflow & Commits**](./Workflow/README.md): Especificación de ramas (`main`, `develop`, `feature/*`, `bugfix/*`, `release/*`, `hotfix/*`), política obligatoria de Pull Requests hacia `main` y convención semántica de commits.
* [**Diagrama Visual de Git Workflow**](./Workflow/diagrama-git-workflow.png): Esquema ilustrativo del ciclo de vida y ramificación del proyecto.
