# Technical Specification — Open Catalog by Templates (Team 09 Market)
## Configurable Item Templates & Professor Course Curation (No Stock Limit)

> 🖥️ **Herramientas visuales interactivas disponibles:**
> - [**Curación de Catálogo del Profesor (UI dedicada)**](file:///C:/Users/totok/Documents/TUP%20-%202026/Programacion/AulaQuest/docs/Mercado/Catalogos/curacion-catalogo-profesor.html) — Creador y gestor de ofertas de cohorte, configuración por plantillas, vista previa del alumno y generador de payloads REST.
> - [**Simulador Integral de Saga y Holds**](file:///C:/Users/totok/Documents/TUP%20-%202026/Programacion/AulaQuest/docs/Mercado/Catalogos/catalogo-abierto-interactivo.html) — Flujo completo de compra, Dual-Hold (Stock + Banco) y contratos Kafka.

---

## 1. Domain Boundary & Architectural Principles

1. **Market as an Orchestrating Storefront / Kiosk:**
   - Market manages **Item Templates**, **Course Cohort Offerings**, and **Purchase Orders**. Catalog offers have **no stock limit** — availability is always unlimited while an offer is active (decision #6).
   - Market **never** persists `student_inventory` (managed exclusively by **Grupo 12**, Bank's own team, since decision #13 — this is not a separate microservice).
   - Market **never** holds coin balances (managed exclusively by **Bank / Team 08**).
2. **Open Catalog with Professor-Driven Configuration (Templates, Not Fixed Items):**
   - Pricing and operational constraints are **not governed by Backoffice**.
   - Professors select an **item template TYPE** (`SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE` — a closed set of types, decision #5) and configure its parameters per cohort: `coinPrice` (within the template's allowed range), `charges`, `applicableChallenges`, `multiplier`, `mode` (`TTL` vs `PER_EXAM`). There are **no fixed tiers and no fixed concrete items** — the professor's configuration *is* the offer (decision #14).
3. **Reserve → Provision → Confirm Saga Compliance:**
   - **Phase 1 (Bank Balance Hold):** Market requests coin reservation $\rightarrow$ Bank locks coins (`HOLD_CREATE_REQUESTED` $\rightarrow$ `HOLD_CREATED`).
   - **Phase 2 (Item Provisioning via Grupo 12):** Market requests acreditación of the asset from **Grupo 12** (`ITEM_PROVISION_REQUESTED` $\rightarrow$ `ITEM_PROVISIONED`).
   - **Phase 3 (Commit & Settlement):** Market orders Bank to finalize the ledger deduction (`HOLD_CONFIRM_REQUESTED` $\rightarrow$ `HOLD_CONFIRMED`) only once item provisioning has been corroborated (decision #8).
   - **Compensation:** If Bank rejects funds or Grupo 12 fails to provision the item, the coin hold is released (`HOLD_RELEASE_REQUESTED`) and the order is marked `CANCELADA` — no stock to restore.

---

## 2. Availability & Concurrency Strategy

### 2.1 No Stock Limit (Decision #6)

Catalog offers have **unlimited availability while active** — there is no `stock`/`availableStock` field, no atomic decrement, and no `stock_holds` table. Concurrency control for a purchase is entirely about the student's **coin balance**, which is Bank's responsibility, not Market's:

* Under 120 concurrent sessions, the only race condition that matters is over-committing an alumno's balance across simultaneous purchases — Bank's `BalanceHold` mechanism already handles that (Section 4 and `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md`).
* Market's only remaining validation before requesting a hold is that the offer itself is **active** (not deactivated/archived); there is no concept of "sold out."

### 2.2 When Market Still Returns 409

`409 Conflict` is only returned for **offer-state** reasons, never for stock exhaustion:

* Offer is inactive or was deleted (logical deletion, RF-NFR-01).
* Offer does not belong to the requested course-cohort.
* Cohort is archived or the student is unenrolled (decision #10 — same treatment as archived).

---

## 3. Open Catalog Parameter Matrix

### 3.1 Item Template Types (Configured by the Professor, Not Fixed by Market)

Market defines a **closed set of template TYPES** (decision #5); it does **not** supply fixed concrete items with a default name/description/icon/tier. Each professor instantiates a `CourseCatalogOffer` from a template type and configures its parameters within the allowed ranges for their cohort (decision #14) — there is no `tpl-shield-base`-style catalog of pre-built items:

| Item Type | What It Does | Professor-Configurable Parameters |
| :--- | :--- | :--- |
| `SHIELD` | Absorbs failed test attempts in code/challenges. | `coinPrice` (within configurable range), `charges`, `applicableChallenges` |
| `BOOST_XP` | Multiplies XP gained upon approved challenges. | `coinPrice`, `multiplier`, `mode`, `durationMinutes`, `attempts`, `consumptionRule` |
| `BOOST_COINS` | Multiplies coin rewards earned from deliveries. | `coinPrice`, `multiplier`, `mode`, `durationMinutes`, `attempts`, `consumptionRule` |
| `LIFE` | Restores additional student life attempts. | `coinPrice`, `livesGranted` (default 1) |

There is no `stock` parameter for any type (decision #6 — no stock limit, ever).

---

### 3.2 Configurable Attributes by Item Type

#### A. Shields (`SHIELD`)
* **`coinPrice`:** Integer $\ge 1$.
* **`charges`:** Integer $\ge 1$ (absorbs up to $N$ failed test attempts before depletion).
* **`applicableChallenges`:**
  - `ALL`: Applies across all challenge modalities (quizzes, practicals, exams).
  - `THEORETICAL_ONLY`: Active only during conceptual quizzes.
  - `PRACTICAL_ONLY`: Active only during programming/IDE code challenges.
  - `NO_EXAMS`: Active for both theoretical quizzes and practical programming exercises; automatically disabled during midterms, finals, or exam mode (Teórico y Práctico, sin efecto en exámenes).

#### B. Boosts (`BOOST_XP`, `BOOST_COINS`)
* **`coinPrice`:** Integer $\ge 1$.
* **`multiplier`:** Decimal scale factor (e.g., `1.25` for +25%, `1.50` for +50%, `2.00` for 2x reward).
* **`mode`:**
  - `TTL` (Time-To-Live countdown):
    - `durationMinutes`: Active duration in minutes (e.g., 60, 120, 240). Activates a running timer upon activation.
  - `PER_EXAM` (Delivery attempt pool):
    - `attempts`: Number of covered evaluations (e.g., 1, 3, 5).
    - `consumptionRule`:
      - `ALWAYS_CONSUME`: Deducts an attempt regardless of whether the student passes or fails.
      - `CONSUME_ON_PASS_ONLY`: Deducts an attempt strictly when the student passes the evaluation.

#### C. Lives (`LIFE`)
* **`coinPrice`:** Integer $\ge 1$.
* **`livesGranted`:** Integer (default: 1). Provisioned into the student's backpack via Grupo 12.

---

## 4. REST API Endpoints (Spring Boot)

### 4.1 Professor Management Endpoints (`ROLE_PROFESSOR`)

#### 1. List Available Template Types
```http
GET /api/v1/market/templates
Authorization: Bearer <JWT>
```
Returns the closed set of item template **types** (`SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE`) and each type's configurable parameter ranges — not a list of fixed concrete items (decision #14).

#### 2. Get Cohort Catalog Configuration
```http
GET /api/v1/market/courses/{courseId}/catalog/manage
Authorization: Bearer <JWT>
X-Roles: ROLE_PROFESSOR
```

#### 3. Publish New Catalog Offer in Course
```http
POST /api/v1/market/courses/{courseId}/catalog/items
Content-Type: application/json

{
  "templateId": "tpl-shield-base",
  "customName": "Advanced Lab Shield",
  "customDescription": "Absorbs up to 2 test failures in practical programming exercises.",
  "coinPrice": 350,
  "active": true,
  "configuration": {
    "itemType": "SHIELD",
    "charges": 2,
    "applicableChallenges": "PRACTICAL_ONLY"
  }
}
```

#### 4. Update Course Catalog Offer
```http
PUT /api/v1/market/courses/{courseId}/catalog/items/{itemId}
Content-Type: application/json

{
  "customName": "Advanced Lab Shield v2",
  "coinPrice": 400,
  "active": true,
  "configuration": {
    "itemType": "SHIELD",
    "charges": 3,
    "applicableChallenges": "NO_EXAMS"
  }
}
```

#### 5. Deactivate Course Catalog Offer (Logical Deletion)
```http
DELETE /api/v1/market/courses/{courseId}/catalog/items/{itemId}
```

---

### 4.2 Student Storefront Endpoints (`ROLE_STUDENT`)

#### 1. Browse Course Storefront
```http
GET /api/v1/market/courses/{courseId}/catalog
Authorization: Bearer <JWT>
X-Roles: ROLE_STUDENT
```

#### 2. Checkout Purchase Order
```http
POST /api/v1/market/orders
Content-Type: application/json

{
  "offerId": "item-course-9912",
  "courseId": "COURSE_PROG4_2026"
}
```
* **Success:** `202 Accepted { "orderId": "ord-88391a", "status": "PROCESSING" }`
* **Offer Not Available:** `409 Conflict { "error": "OFFER_NOT_AVAILABLE", "message": "The selected offer is inactive or does not belong to this cohort." }` (no longer stock-related — decision #6; offers are never sold out)

---

## 5. Kafka Provisioning Payload Transferred to Grupo 12

Once Bank confirms the coin hold on `bank.holds.events` (`HOLD_CREATED`), Market dispatches the provisioning command to **Grupo 12** on `inventory.items.commands`:

```json
{
  "eventId": "c32f94d6-9a30-4b24-ae33-14daf13e2203",
  "eventType": "ITEM_PROVISION_REQUESTED",
  "timestamp": "2026-09-16T22:40:03Z",
  "producer": "team-09-market",
  "payload": {
    "orderId": "ord-88391a",
    "holdId": "hld-99201",
    "studentId": "usr-4821",
    "courseId": "COURSE_PROG4_2026",
    "itemPayload": {
      "catalogItemId": "item-course-9912",
      "templateId": "tpl-shield-base",
      "itemType": "SHIELD",
      "name": "Advanced Lab Shield",
      "icon": "shield-rune",
      "properties": {
        "chargesTotal": 2,
        "chargesRemaining": 2,
        "applicableChallenges": "PRACTICAL_ONLY"
      }
    }
  }
}
```
