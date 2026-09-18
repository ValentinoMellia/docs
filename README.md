# Technical & Architectural Documentation — Market (Team 09)
## Distributed Gamified Platform · Aula Quest (TUP UTN FRC)

This repository maintains the technical specifications, domain models, **Open Catalog** design, **Auction Subsystem** architecture, and transactional saga protocols between **Market (Team 09)** and **Bank (Team 08 / Grupo 12)** — which, since decision #13, also owns the student's backpack/inventory.

---

## 🏛️ Bounded Context & Platform Architecture Doctrine

1. **Market as an Orchestrating Storefront / Kiosk:**
   - Sole authority over **Course Catalog Offerings**, **Open Catalog Curation**, **Purchase Orders**, and **Auctions**. Offers have unlimited availability while active — no stock limit (decision #6).
   - **Does not** persist student backpacks (`student_inventory`) — owned by **Grupo 12** (Bank's team) since decision #13.
   - **Does not** manage coin balances (governed by Bank).
   - **Does not** evaluate challenge effects or charge deductions (exclusive contract between Motor de Desafíos and Grupo 12, decision #11).
2. **Reserve → Provision → Confirm Saga (PRD Compliance):**
   - **Money is never deducted before item delivery is corroborated.**
   - Sequence:
     1. **Phase 1 (Coin Hold):** Market requests coin reservation $\rightarrow$ Bank locks coins (`HOLD_CREATE_REQUESTED` $\rightarrow$ `HOLD_CREATED`).
     2. **Phase 2 (Provision via Grupo 12):** Market requests acreditación of the asset from **Grupo 12** (`ITEM_PROVISION_REQUESTED` $\rightarrow$ `ITEM_PROVISIONED`).
     3. **Phase 3 (Commit & Debit):** Market authorizes Bank to finalize the ledger deduction (`HOLD_CONFIRM_REQUESTED` $\rightarrow$ `HOLD_CONFIRMED`) only once item provisioning has been corroborated (decision #8).
3. **Open Catalog with Free Market Pricing:**
   - Item prices and operational constraints are **not governed by Backoffice**.
   - Market defines a closed set of **Item Template types** (`SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE`). There are no fixed tiers or fixed concrete items — professors freely configure `coinPrice`, `charges`, `applicableChallenges`, `multiplier`, `mode` (`TTL` vs `PER_EXAM`) per cohort on top of a chosen template type. Availability is always unlimited while an offer is active — there is no stock limit.
4. **Everything is an Item Doctrine:**
   - Shields, boosts, and **Lives** are uniformly modeled as items in the catalog and provisioned into the student's backpack via Grupo 12.

---

## 🗂️ Repository Structure

```
docs/
├── Comunicacion/
│   └── Grupo-08-Banco/                      # Bank integration & Two-Phase Hold saga
│       └── flujo-comunicacion-banco.md      # Complete English Kafka events & dual-hold saga contract
│
├── Mercado/                                 # Market domain specifications
│   ├── Catalogos/                           # Open Catalog by configurable templates
│   │   ├── README.md                        # Technical specification, template parameters, DTOs & payloads
│   │   └── catalogo-abierto-interactivo.html# Interactive simulator & dual-hold saga previewer
│   │
│   └── Subastas/                            # Auction subsystem (Epic E-07)
│       ├── README.md                        # Architecture index
│       ├── documento-arquitectura-subastas.html # Master document with embedded Archify viewer
│       ├── 01-analisis-opciones-arquitectura.md # Architectural alternatives & trade-offs
│       ├── 02-matriz-fallos-resiliencia-y-soluciones.md # Concurrency, locks & tie-breakers
│       ├── 03-contratos-eventos-e-idempotencia.md       # Event idempotency & deduplication
│       ├── flujo-subasta-archify.html       # Navigable Archify sequence diagram
│       └── flujo-subasta-archify.json       # JSON specification
│
├── Workflow/                                # Development & Git workflow
│   ├── README.md                            # Branching & Pull Request policies
│   └── diagrama-git-workflow.png            # Visual branch lifecycle diagram
│
├── CONTEXTO-MERCADO-SPRINT1.md              # Consolidated team context for Sprint 1
├── diagramas-mercado.md                     # Mermaid diagrams (Context, Domain, Saga Machines)
├── PRD-Plataforma-Gamificada-TP.pdf         # Official product requirements document
└── Sprint0_Propuesta_Mercado.pdf            # Sprint 0 initial team proposal
```

---

## 📡 Kafka Topics & Microservice Interconnects

All topics, commands, and events adhere to standard English naming:

| Topic Name | Message Semantics | Producers | Consumers |
| :--- | :--- | :--- | :--- |
| `bank.holds.commands` | Commands to create, confirm, or release coin balance holds | `team-09-market` | `team-08-bank` |
| `bank.holds.events` | Factual lifecycle events of balance holds (`HOLD_CREATED`, `HOLD_CONFIRMED`, `HOLD_RELEASED`, `HOLD_REJECTED`) | `team-08-bank` | `team-09-market`, `team-11-notifications` |
| `inventory.items.commands` | Commands to credit assets to student backpacks | `team-09-market` | `inventory-service` |
| `inventory.items.events` | Factual events confirming asset delivery (`ITEM_PROVISIONED`, `ITEM_PROVISION_FAILED`) | `inventory-service` | `team-09-market` |
| `market.orders.events` | Factual order lifecycle events | `team-09-market` | `team-11-notifications` |
| `market.auctions.events` | Factual auction lifecycle events (`AUCTION_OPENED`, `AUCTION_CLOSED`, `AUCTION_CANCELLED`, `AUCTION_AWARDED`) | `team-09-market` | `team-11-notifications`, `team-08-bank`, `team-02-courses` |
| `notifications.alerts` | Real-time alerts (e.g. `BID_OUTBID`) for immediate student notification | `team-09-market` | `team-11-notifications` |

---

## 🏪 Key Deliverables & Interactive Tools

* [**Open Catalog Technical Specification**](./Mercado/Catalogos/README.md): Definition of item template types, professor customization parameters (no stock limit), and REST DTOs.
* [**Open Catalog & Dual-Hold Interactive Simulator**](./Mercado/Catalogos/catalogo-abierto-interactivo.html): Web-based visualizer for live template customization and saga JSON payload inspection.
* [**Bank & Grupo 12 Saga Protocol**](./Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md): Complete specification of the coin-hold + item-provisioning transaction.
* [**Architectural Diagrams (Mermaid)**](./diagramas-mercado.md): C4 context, domain model by templates (no stock), state machines, and auction settlement sequence.
* [**Sprint 1 Master Context**](./CONTEXTO-MERCADO-SPRINT1.md): Consolidated background, team decisions, and Sprint 1 planning notes.
