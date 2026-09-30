# Technical & Architectural Documentation — Market (Team 09)
## Distributed Gamified Platform · Aula Quest (TUP UTN FRC)

This repository maintains the technical specifications, domain models, **Open Catalog** design, **Auction Subsystem** architecture, and transactional saga protocols between **Market (Team 09)**, **Bank / Accounting (Team 08)** and **Inventory (Grupo 12)**, plus the Taiga backlog export and the team workflow.

> **Nota de vigencia:** Bank and Inventory are two separate microservices (confirmed with Grupo 08 on 26/09/2026). This corrects decision #13 of `CONTEXTO-MERCADO-SPRINT1.md` §12, which had them merged. See [`ESTADO-IMPLEMENTACION-BANCO.md`](./integracion/banco/ESTADO-IMPLEMENTACION-BANCO.md).

---

## 🏛️ Bounded Context & Platform Architecture Doctrine

1. **Market as an Orchestrating Storefront / Kiosk:**
   - Sole authority over **Course Catalog Offerings**, **Open Catalog Curation**, **Purchase Orders**, and **Auctions**. Offers have an optional `stock` field, configured per course-cohort by the professor — unset means unlimited availability, a positive integer sets a finite cap (decision #6, revised 19/09). Does not apply to auctions.
   - **Does not** persist student backpacks (`student_inventory`) — owned by **Inventory (Grupo 12)**.
   - **Does not** manage coin balances (governed by Bank).
   - **Does not** evaluate challenge effects or charge deductions (exclusive contract between Motor de Desafíos and Grupo 12, decision #11).
2. **Reserve → Provision → Confirm Saga (PRD Compliance):**
   - **Money is never deducted before item delivery is corroborated.**
   - Sequence:
     1. **Phase 1 (Coin Hold):** Market requests coin reservation → Bank locks coins (`HOLD_CREATE_REQUESTED` → `HOLD_CREATED`).
     2. **Phase 2 (Provision via Inventory):** Market requests crediting of the asset from **Inventory (Grupo 12)** (`ITEM_PROVISION_REQUESTED` → `ITEM_PROVISIONED`).
     3. **Phase 3 (Commit & Debit):** Market authorizes Bank to finalize the ledger deduction (`HOLD_CONFIRM_REQUESTED` → `HOLD_CONFIRMED`) only once item provisioning has been corroborated (decision #8).
3. **Open Catalog with Free Market Pricing:**
   - Item prices and operational constraints are **not governed by Backoffice**.
   - Market defines a closed set of **Item Template types** (`SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE`). There are no fixed tiers or fixed concrete items — professors freely configure `coinPrice`, `charges`, `applicableChallenges`, `multiplier`, `mode` (`TTL` vs `PER_EXAM`) per cohort on top of a chosen template type. Availability is unlimited while an offer is active unless the professor configures a finite `stock` cap for it (decision #6, revised 19/09).
4. **Everything is an Item Doctrine:**
   - Shields, boosts, and **Lives** are uniformly modeled as items in the catalog and provisioned into the student's backpack via Inventory.

---

## 📍 Fuente de Verdad por Tema

> Antes de citar cualquier documento de este repo, verificá acá cuál es el vigente. Los demás pueden estar deprecados, ser históricos, o cubrir solo un recorte.

| Tema | Documento vigente | Otros documentos relacionados |
|---|---|---|
| **Estado actual de Mercado** (compra, gestión de tienda, subasta) | [`mercado/README.md`](./mercado/README.md) → [`estado-actual/`](./mercado/estado-actual/brechas-y-pendientes.md) | Verificado contra el código (29/09/2026); **manda sobre los documentos de diseño** cuando difieren |
| **Accounting (Banco): estado del repo y contratos respecto de Mercado** | [`integracion/banco/accounting-estado-y-contratos.md`](./integracion/banco/accounting-estado-y-contratos.md) | Verificado contra `tpi-accounting` develop @ 3013f6c (30/09/2026) |
| **Integración Mercado ↔ Accounting** (desajustes y flujo recomendado) | [`integracion/banco/estado-integracion-mercado-accounting.md`](./integracion/banco/estado-integracion-mercado-accounting.md) | Reemplaza como estado a `ESTADO-IMPLEMENTACION-BANCO.md` |
| Contexto y decisiones del equipo (Sprint 1) | [`arquitectura/CONTEXTO-MERCADO-SPRINT1.md`](./arquitectura/CONTEXTO-MERCADO-SPRINT1.md) | Su §13 es un registro histórico de auditoría, no un índice de vigencia. |
| Catálogo abierto por plantillas | [`mercado/catalogos/README.md`](./mercado/catalogos/README.md) | — |
| Compra directa (contratos REST/eventos) | [`arquitectura/CONTRATOS-COMUNICACION-SPRINT1.md`](./arquitectura/CONTRATOS-COMUNICACION-SPRINT1.md) | Deriva de `CONTEXTO-MERCADO-SPRINT1.md` §7/§9/§10 |
| Integración Mercado ↔ Banco ↔ Inventario (diseño objetivo) | [`integracion/banco/flujo-mercado-inventario.md`](./integracion/banco/flujo-mercado-inventario.md) | [`archivado/flujo-comunicacion-banco.md`](./integracion/banco/archivado/flujo-comunicacion-banco.md) — **deprecado**, predecesor histórico. [`contrato-integracion-mercado-accounting.md`](./integracion/banco/contrato-integracion-mercado-accounting.md) — contrato Kafka propuesto para T07, aún no confirmado contra el código (ver estado real). |
| Integración con Banco — estado real de implementación | [`integracion/banco/ESTADO-IMPLEMENTACION-BANCO.md`](./integracion/banco/ESTADO-IMPLEMENTACION-BANCO.md) | Refleja lo implementado en `tpi-market` (T03/T04 hechos, sincrónico y mockeado; T05/T07 sin iniciar) — distinto del diseño objetivo de la fila anterior |
| Subastas — arquitectura y resiliencia | [`mercado/subastas/README.md`](./mercado/subastas/README.md) (índice) → `01-analisis-opciones-arquitectura.md`, `02-matriz-fallos-resiliencia-y-soluciones.md` | Vigentes desde el commit `f1aca8b` (18/09). **Diseño de Fase 3: no hay código de subastas en `tpi-market` todavía.** |
| Subastas — contratos de eventos e idempotencia | [`mercado/subastas/03-contratos-eventos-e-idempotencia.md`](./mercado/subastas/03-contratos-eventos-e-idempotencia.md) | — |
| **Meta colectiva** (aporte grupal con pérdida si falla) | [`mercado/metas-colectivas/README.md`](./mercado/metas-colectivas/README.md) | Diseño, sin código. Acreditar el mismo ítem a varios alumnos queda **pendiente de Banco** (§9) |
| Propuesta de ítems nuevos (cofres, cosméticos, ayudas) | [`mercado/nuevos-items/propuesta-cofres-metas-y-nuevos-items.md`](./mercado/nuevos-items/propuesta-cofres-metas-y-nuevos-items.md) | Propuesta sin acordar; su §3 remite a `metas-colectivas/` |
| Cómo viaja una petición (nginx → Gateway → micros, service tokens, Kafka) | [`arquitectura/flujo-de-una-peticion.md`](./arquitectura/flujo-de-una-peticion.md) | Verificado contra `tpi-api-gateway` y `tpi-system-compose`; incluye inconsistencias abiertas (§7) |
| Estándar de eventos Kafka (envelope, reglas) | [`arquitectura/KAFKA_EVENT_STANDARD.md`](./arquitectura/KAFKA_EVENT_STANDARD.md) | La lista de topics **provisionados** vive en `tpi-system-compose` (ver sección Kafka abajo) |
| Diagramas de dominio (Mermaid) | [`arquitectura/diagramas-mercado.md`](./arquitectura/diagramas-mercado.md) | — |
| Workflow de Git / PRs | [`gestion/workflow/README.md`](./gestion/workflow/README.md) | — |
| Backlog Taiga — plan de actualización | [`gestion/taiga/PLAN-actualizacion-backlog-sprint1.md`](./gestion/taiga/PLAN-actualizacion-backlog-sprint1.md) | Ver headers de estado en `EPIC-131`, `EPIC-482`, `EPIC-770`, `US-810` |

---

## 🗂️ Repository Structure

```
docs/
├── producto/                                # What we were asked to build
│   ├── PRD-Plataforma-Gamificada-TP.pdf     # Official product requirements document
│   ├── Sprint0_Propuesta_Mercado.pdf        # Sprint 0 initial team proposal
│   ├── propuesta-mercado-enriquecedor.html  # Market enrichment proposal
│   └── catalogo-items-consumibles.html      # Consumable items catalogue
│
├── arquitectura/                            # Cross-cutting design & contracts
│   ├── CONTEXTO-MERCADO-SPRINT1.md          # Consolidated team context for Sprint 1
│   ├── CONTRATOS-COMUNICACION-SPRINT1.md    # Catalogue + direct purchase REST/event contracts
│   ├── KAFKA_EVENT_STANDARD.md              # Platform-wide Kafka event standard
│   ├── flujo-de-una-peticion.md             # Request path: nginx → Gateway → micros, service tokens, Kafka
│   ├── diagramas-mercado.md                 # Mermaid diagrams (Context, Domain, Saga Machines)
│   └── Transacciones.docx                   # Transactions notes
│
├── integracion/
│   └── banco/                               # Bank / Accounting / Inventory integration
│       ├── accounting-estado-y-contratos.md # Accounting repo and contracts as seen from Market
│       ├── estado-integracion-mercado-accounting.md # Real Market ↔ Accounting status (read first)
│       ├── ESTADO-IMPLEMENTACION-BANCO.md   # Historical status (27/09), superseded
│       ├── flujo-mercado-inventario.md      # Target saga design (current)
│       ├── contrato-integracion-mercado-accounting.md # Proposed Kafka contract (T07)
│       ├── Banco-T08_Mercado-T09_Documento-de-integracion.docx
│       └── archivado/
│           └── flujo-comunicacion-banco.md  # DEPRECATED predecessor
│
├── mercado/                                 # Market domain specifications
│   ├── README.md                            # Market overview and status by function
│   ├── estado-actual/                       # Code-verified current state (read first)
│   │   ├── compra.md · gestion-de-tienda.md · subasta.md
│   │   └── brechas-y-pendientes.md          # Prioritized gaps
│   ├── catalogos/                           # Open Catalog by configurable templates
│   │   ├── README.md                        # Technical specification, DTOs & payloads
│   │   ├── catalogo-abierto-interactivo.html# Interactive simulator & dual-hold saga previewer
│   │   └── curacion-catalogo-profesor.html  # Professor curation UI mock
│   ├── metas-colectivas/                    # Collective goal ("Colecta") — design only
│   │   └── README.md                        # Rules, states, API, pending items for Bank
│   ├── nuevos-items/                        # New item proposals (chests, cosmetics, helpers)
│   │   └── propuesta-cofres-metas-y-nuevos-items.md
│   └── subastas/                            # Auction subsystem (Epic #577, Phase 3 — design only)
│       ├── README.md                        # Architecture index
│       ├── 01-analisis-opciones-arquitectura.md
│       ├── 02-matriz-fallos-resiliencia-y-soluciones.md
│       ├── 03-contratos-eventos-e-idempotencia.md
│       ├── documento-arquitectura-subastas.html # Master document with embedded Archify viewer
│       ├── flujo-subasta-archify.html       # Navigable Archify sequence diagram
│       └── flujo-subasta-archify.json       # JSON specification
│
└── gestion/                                 # Project management
    ├── taiga/                               # Taiga backlog export (epics + user stories)
    └── workflow/                            # Branching & Pull Request policies + diagram
```

---

## 📡 Kafka Topics

The platform provisions **one domain topic per team plus its `.DLT`**, with 3 partitions and replication factor 1 (auto-creation is disabled). The operational source of truth is `event-bus/init-topics.sh` and `docs/kafka-contract.md` in the `tpi-system-compose` repository. Relevant to Market:

| Topic | Purpose | Notes |
| :--- | :--- | :--- |
| `market.events` (+ `.DLT`) | Market domain events (orders, catalogue, auctions) | Producer: Market |
| `accounting.events` (+ `.DLT`) | Bank / Accounting domain events (holds, settlement) | Producer: Bank |
| `notifications.events` (+ `.DLT`) | Student-facing notifications | Consumer of Market / Bank events |
| `courses.events`, `users.events` (+ `.DLT`) | Course lifecycle and user lifecycle | Consumed for cohort / user validation |

The per-flow topic names used in earlier drafts (`bank.holds.commands`, `bank.holds.events`, `inventory.items.commands`, `inventory.items.events`, `market.orders.events`, `market.auctions.events`, `notifications.alerts`, `accounting.settlement.*`) are **design proposals, not provisioned topics**. Event types travel inside the standard envelope defined in [`KAFKA_EVENT_STANDARD.md`](./arquitectura/KAFKA_EVENT_STANDARD.md). `tpi-market` implements a Kafka transport (transactional outbox, idempotent consumers) but still uses its own topic names (`accounting.holds.*`, `inventory.items.*`, `market.orders.events`) that are not provisioned; see [`estado-integracion-mercado-accounting.md`](./integracion/banco/estado-integracion-mercado-accounting.md).

---

## 🏪 Key Deliverables & Interactive Tools

* [**Open Catalog Technical Specification**](./mercado/catalogos/README.md): Definition of item template types, professor customization parameters (optional per-offer stock), and REST DTOs.
* [**Open Catalog & Dual-Hold Interactive Simulator**](./mercado/catalogos/catalogo-abierto-interactivo.html): Web-based visualizer for live template customization and saga JSON payload inspection.
* [**Bank & Inventory Saga Protocol**](./integracion/banco/flujo-mercado-inventario.md): Complete specification of the coin-hold + item-provisioning transaction. (`archivado/flujo-comunicacion-banco.md` is the deprecated predecessor.)
* [**Architectural Diagrams (Mermaid)**](./arquitectura/diagramas-mercado.md): C4 context, domain model by templates (optional stock), state machines, and auction settlement sequence.
* [**Sprint 1 Master Context**](./arquitectura/CONTEXTO-MERCADO-SPRINT1.md): Consolidated background, team decisions, and Sprint 1 planning notes.
