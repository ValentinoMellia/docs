# Technical Specification — Open Catalog & Stock Holds (Team 09 Market)
## Base Item Templates, Professor Course Curation & Local Stock Hold (Fail-Fast)

> 🖥️ **Herramientas visuales interactivas disponibles:**
> - [**Curación de Catálogo del Profesor (UI dedicada)**](file:///C:/Users/totok/Documents/TUP%20-%202026/Programacion/AulaQuest/docs/Mercado/Catalogos/curacion-catalogo-profesor.html) — Creador y gestor de ofertas de cohorte, configuración por plantillas, vista previa del alumno y generador de payloads REST.
> - [**Simulador Integral de Saga y Holds**](file:///C:/Users/totok/Documents/TUP%20-%202026/Programacion/AulaQuest/docs/Mercado/Catalogos/catalogo-abierto-interactivo.html) — Flujo completo de compra, Dual-Hold (Stock + Banco) y contratos Kafka.

---

## 1. Domain Boundary & Architectural Principles

1. **Market as an Orchestrating Storefront / Kiosk:**
   - Market manages **Base Item Templates**, **Course Cohort Offerings**, and **Local Stock Holds**.
   - Market **never** persists `student_inventory` (managed exclusively by the **Inventory Microservice**).
   - Market **never** holds coin balances (managed exclusively by **Bank / Team 08**).
2. **Open Catalog with Professor-Driven Pricing & Optional Stock:**
   - Pricing and operational constraints are **not governed by Backoffice**.
   - Professors select a base template and customize all attributes: `coinPrice`, `charges`, `applicableChallenges`, `multiplier`, `mode` (`TTL` vs `PER_EXAM`), and optional finite cohort **`stock`**.
3. **Local Stock Hold (Fail-Fast Pattern):**
   - If an offer has limited stock (`stock > 0`), Market executes an **atomic stock reservation (`StockHold`) locally before touching Kafka or Bank**.
   - If stock is exhausted, Market immediately rejects the request in 2ms with `HTTP 409 Conflict (OUT_OF_STOCK)`. This prevents over-selling and protects Kafka and Bank from processing unfulfillable orders.
4. **Two-Phase Hold Saga Compliance:**
   - **Phase 0 (Stock Hold):** Market locks local stock unit (`StockHold` status: `PENDING`, TTL: 5 min).
   - **Phase 1 (Bank Balance Hold):** Market requests coin reservation $\rightarrow$ Bank locks coins (`HOLD_CREATE_REQUESTED` $\rightarrow$ `HOLD_CREATED`).
   - **Phase 2 (Inventory Provisioning):** Market orders the **Inventory Microservice** to credit the asset (`ITEM_PROVISION_REQUESTED` $\rightarrow$ `ITEM_PROVISIONED`).
   - **Phase 3 (Commit & Settlement):** Market orders Bank to finalize the ledger deduction (`HOLD_CONFIRM_REQUESTED` $\rightarrow$ `HOLD_CONFIRMED`) and commits the stock hold (`StockHold` status: `COMMITTED`).
   - **Mirrored Compensation:** If Bank rejects funds or Inventory crashes, both holds (the coin hold in Bank and the stock hold in Market) are released simultaneously.

---

## 2. Stock Management & Concurrency Strategy

### 2.1 Optional Stock Modalities

* **Unlimited Availability (`stock IS NULL`):**
  - Common consumables (e.g. basic potions, apprentice shields).
  - Bypasses stock locks completely to avoid database contention.
* **Finite Cohort Pool (`stock > 0`):**
  - Rare items, high-tier buffs, or exam-specific limited equipment.
  - Enforces strict concurrency control.

### 2.2 Atomic Concurrency at Database Level

To prevent race conditions and over-selling under 120 concurrent sessions:

```sql
-- Atomic Decrement Execution in PostgreSQL
UPDATE course_catalog_offers 
SET available_stock = available_stock - 1, 
    updated_at = NOW()
WHERE id = :offerId 
  AND available_stock > 0;
```
* If affected rows == `1`: Stock was secured. Market inserts a `StockHold` record:
  ```sql
  INSERT INTO stock_holds (id, catalog_offer_id, purchase_order_id, student_id, status, expires_at, created_at)
  VALUES (:holdId, :offerId, :orderId, :studentId, 'PENDING', NOW() + INTERVAL '5 minutes', NOW());
  ```
* If affected rows == `0`: Sold out. Returns immediate `HTTP 409 Conflict`.

### 2.3 Stock Hold Lifecycle & States

* **`PENDING`:** Temporary reservation active while the Kafka saga runs (TTL: 5 minutes).
* **`COMMITTED`:** Bank confirmed debit (`HOLD_CONFIRMED`). Stock unit permanently consumed.
* **`RELEASED`:** Bank rejected funds or Inventory provisioning failed. Stock unit returned to catalog (`available_stock = available_stock + 1`).
* **`EXPIRED`:** Saga did not complete within the 5-minute TTL. An asynchronous `@Scheduled` reconciliation job releases the hold automatically.

---

## 3. Open Catalog Parameter Matrix

### 3.1 Base Templates (Supplied by Market)

| Template ID | Item Type | Default Name | Description | Default Icon | Professor-Configurable Parameters |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `tpl-shield-base` | `SHIELD` | Defense Aegis | Absorbs failed test attempts in code/challenges. | `shield-rune` | `coinPrice`, `stock`, `charges`, `applicableChallenges` |
| `tpl-boost-xp` | `BOOST_XP` | Experience Elixir | Multiplies XP gained upon approved challenges. | `potion-violet` | `coinPrice`, `stock`, `multiplier`, `mode`, `durationMinutes`, `attempts`, `consumptionRule` |
| `tpl-boost-coins` | `BOOST_COINS` | Coin Magnet | Multiplies coin rewards earned from deliveries. | `coin-magnet` | `coinPrice`, `stock`, `multiplier`, `mode`, `durationMinutes`, `attempts`, `consumptionRule` |
| `tpl-life-potion` | `LIFE` | Revive Flask | Restores additional student life attempts. | `heart-flask` | `coinPrice`, `stock`, `livesGranted` (default 1) |

---

### 3.2 Configurable Attributes by Item Type

#### A. Shields (`SHIELD`)
* **`coinPrice`:** Integer $\ge 1$.
* **`stock`:** Optional integer (null = unlimited, $>0$ = finite pool).
* **`charges`:** Integer $\ge 1$ (absorbs up to $N$ failed test attempts before depletion).
* **`applicableChallenges`:**
  - `ALL`: Applies across all challenge modalities (quizzes, practicals, exams).
  - `THEORETICAL_ONLY`: Active only during conceptual quizzes.
  - `PRACTICAL_ONLY`: Active only during programming/IDE code challenges.
  - `NO_EXAMS`: Active for both theoretical quizzes and practical programming exercises; automatically disabled during midterms, finals, or exam mode (Teórico y Práctico, sin efecto en exámenes).

#### B. Boosts (`BOOST_XP`, `BOOST_COINS`)
* **`coinPrice`:** Integer $\ge 1$.
* **`stock`:** Optional integer.
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
* **`stock`:** Optional integer.
* **`livesGranted`:** Integer (default: 1). Provisioned into student inventory.

---

## 4. REST API Endpoints (Spring Boot)

### 4.1 Professor Management Endpoints (`ROLE_PROFESSOR`)

#### 1. Retrieve Base Templates
```http
GET /api/v1/market/templates
Authorization: Bearer <JWT>
```

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
  "stock": 10,
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
  "stock": 5,
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
* **Success:** `202 Accepted { "orderId": "ord-88391a", "status": "PROCESSING", "stockHoldId": "stk-hld-001" }`
* **Out of Stock:** `409 Conflict { "error": "OUT_OF_STOCK", "message": "The selected item is no longer available in this cohort." }`

---

## 5. Kafka Provisioning Payload Transferred to Inventory Microservice

Once Bank confirms the coin hold on `bank.holds.events` (`HOLD_CREATED`), Market dispatches the provisioning command to **Inventory** on `inventory.items.commands`:

```json
{
  "eventId": "c32f94d6-9a30-4b24-ae33-14daf13e2203",
  "eventType": "ITEM_PROVISION_REQUESTED",
  "timestamp": "2026-09-16T22:40:03Z",
  "producer": "team-09-market",
  "payload": {
    "orderId": "ord-88391a",
    "stockHoldId": "stk-hld-001",
    "bankHoldId": "hld-99201",
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
