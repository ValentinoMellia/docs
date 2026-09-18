# Architectural Diagrams — Market (Team 09)
## Distributed Gamified Platform · Aula Quest (TUP UTN FRC)

---

## 1. Context Diagram: Market & External Microservices

Market acts as an orchestrating Kiosk / Storefront. In accordance with platform design and the Reserve → Provision → Confirm Saga (no local stock — decision #6: catalog offers have unlimited availability while active):
- **Market (Team 09):** Orchestrates the purchase/auction saga and curates the Open Catalog (templates with professor-configured parameters, no fixed stock).
- **Bank / Grupo 12 (Team 08):** Manages coin balances, handles `BalanceHold` commands, settles ledger debits, and — since decision #13 — also owns the student backpack (`student_inventory`), its lifecycle, and runtime effect execution. This is not a separate microservice; Grupo 12 is Bank's own Taiga team.

```mermaid
graph TB
    subgraph Web Clients
        Student[Student Frontend - Storefront / Auctions]
        Professor[Professor Frontend - Catalog Curation / Auctions]
    end

    GW[API Gateway / Perimetral Security]
    Student --> GW
    Professor --> GW

    GW -->|HTTP REST| T09[Team 09 · Market<br/><b>Open Catalog / Direct Purchase / Auctions</b>]

    subgraph Kafka Event Bus Ecosystem
        T09 -->|bank.holds.commands| BUS[(Apache Kafka<br/>Event Bus)]
        BUS -->|bank.holds.events| T09
        
        T09 -->|inventory.items.commands| BUS
        BUS -->|inventory.items.events| T09

        BUS <-->|Balance Holds, Ledger Debits & Item Provisioning| T08[Team 08 · Bank / Grupo 12<br/><b>Ledger, Coin Holds & Student Backpack</b>]
    end

    BUS -.->|Informative Alerts| T11[Team 11 · Notifications]
```

---

## 2. Market Domain Model (Open Catalog by Templates, No Stock)

Market manages item templates, cohort offerings, purchase orders, and auctions. There is **no local stock** (decision #6: offers have unlimited availability while active) and **no `ItemInventario`** — final item acreditación is an external contract with Grupo 12 (decision #13):

```mermaid
classDiagram
    class ItemTemplate {
        +UUID id
        +ItemType type  // SHIELD | BOOST_XP | BOOST_COINS | LIFE — closed set of TYPES, not fixed concrete items
        +PriceRange configurablePriceRange
        +ConfigurableParams configurableParams  // magnitude, charges, applicable challenges, etc.
        +String effectContract  // name contract shared with Theme 10 / Grupo 12, e.g. ABSORB_FAILURE, XP_MULTIPLIER
        +boolean active
    }

    class CourseCatalogOffer {
        +UUID id
        +UUID courseCohortId
        +UUID itemTemplateId
        +String customName
        +String customDescription
        +int coinPrice  // chosen by professor within the template's configurable price range
        +boolean active
        +ItemConfiguration configuration
        +Instant createdAt
        +Instant updatedAt
    }

    class ShieldConfiguration {
        +int charges
        +ApplicableChallenges scope  // ALL | THEORETICAL_ONLY | PRACTICAL_ONLY | NO_EXAMS
    }

    class BoostConfiguration {
        +double multiplier  // e.g., 1.25, 1.5, 2.0
        +BoostMode mode  // TTL | PER_EXAM
        +Integer durationMinutes  // if mode == TTL
        +Integer attempts  // if mode == PER_EXAM
        +ConsumptionRule consumptionRule  // ALWAYS_CONSUME | CONSUME_ON_PASS_ONLY
    }

    class LifeConfiguration {
        +int livesGranted  // default: 1
    }

    class PurchaseOrder {
        +UUID id
        +UUID courseCohortId
        +UUID studentId
        +UUID catalogOfferId
        +int snapshotPrice
        +UUID holdId  // Bank coin balance lock (unified field name)
        +String idempotencyKey
        +OrderStatus status
        +Instant createdAt
        +Instant completedAt
    }

    class Auction {
        +UUID id
        +UUID courseCohortId
        +UUID catalogOfferId
        +UUID professorId
        +Instant startsAt
        +Instant endsAt
        +Integer minimumBid
        +AuctionStatus status
        +long version
    }

    class Bid {
        +UUID id
        +UUID auctionId
        +UUID studentId
        +int amount
        +UUID holdId  // Locked funds in Bank
        +BidStatus status
        +Instant createdAt
    }

    ItemTemplate "1" --> "0..*" CourseCatalogOffer : instantiates
    CourseCatalogOffer "1" *-- "1" ShieldConfiguration : when type == SHIELD
    CourseCatalogOffer "1" *-- "1" BoostConfiguration : when type == BOOST_*
    CourseCatalogOffer "1" *-- "1" LifeConfiguration : when type == LIFE

    CourseCatalogOffer "1" --> "0..*" PurchaseOrder : transactions
    CourseCatalogOffer "1" --> "0..*" Auction : featured_in
    Auction "1" --> "0..*" Bid : receives

    note for PurchaseOrder "Item acreditación is external: acreditación via Grupo 12 (external) — no ItemInventario in Market's domain"
```

---

## 3. State Machines

### 3.1 Purchase Order State Machine (Reserve → Provision → Confirm Saga)

> Note: there is no local stock step and no "create `ItemInventario`" step — the hold is requested directly from Bank, and item acreditación is requested and corroborated against Grupo 12 before the debit is confirmed (decision #8 + #13).

```mermaid
stateDiagram-v2
    [*] --> CREATED : Student triggers purchase via REST
    CREATED --> HOLD_REQUESTED : Emits HOLD_CREATE_REQUESTED to Bank
    HOLD_REQUESTED --> PROVISIONING_ITEM : Receives HOLD_CREATED from Bank
    HOLD_REQUESTED --> REJECTED_FUNDS : Receives HOLD_REJECTED
    
    PROVISIONING_ITEM --> DEBIT_REQUESTED : Receives ITEM_PROVISIONED from Grupo 12
    PROVISIONING_ITEM --> COMPENSATING : Receives ITEM_PROVISION_FAILED from Grupo 12
    
    COMPENSATING --> CANCELADA : Releases Bank Hold (HOLD_RELEASE_REQUESTED)
    
    DEBIT_REQUESTED --> CONFIRMED : Receives HOLD_CONFIRMED
    
    CONFIRMED --> [*]
    REJECTED_FUNDS --> [*]
    CANCELADA --> [*]
```

---

## 4. Sequence Diagram: Direct Purchase (Reserve → Provision via Grupo 12 → Confirm)

```mermaid
sequenceDiagram
    autonumber
    actor Student as Student (Web Frontend)
    participant GW as API Gateway
    participant Market as Market (Team 09)
    participant BusKafka as Kafka Bus
    participant Bank as Bank (Team 08)
    participant Grupo12 as Grupo 12 (Item Provisioning)

    Student->>GW: POST /api/v1/market/orders {offerId: "item-course-9912", courseId: "COURSE_PROG4_2026"}
    GW->>Market: Forwards request with auth headers (X-User-Id, X-Roles)
    Market->>Market: Validates cohort active, offer active, current price (no stock check — decision #6)
    Market-->>Student: 202 Accepted {orderId: "ord-88391a", status: "PROCESSING"}

    %% Phase 1: Coin Hold in Bank
    rect rgb(245, 243, 255)
    Note over Market,Bank: Phase 1: Reserve Coins in Bank (BalanceHold)
    Market->>BusKafka: bank.holds.commands: HOLD_CREATE_REQUESTED {orderId, studentId, amount: 350}
    
    alt Insufficient Balance in Bank
        Bank-->>BusKafka: bank.holds.events: HOLD_REJECTED {orderId, reason: "INSUFFICIENT_FUNDS"}
        Market-->>Student: SSE: FAILED (INSUFFICIENT_FUNDS)
    else Sufficient Balance in Bank
        Bank->>Bank: Locks 350 coins (status: PENDING, holdId: "hld-99201")
        Bank->>BusKafka: bank.holds.events: HOLD_CREATED {holdId: "hld-99201", status: "PENDING"}
    end
    end

    %% Phase 2: Request item provisioning from Grupo 12
    rect rgb(236, 253, 245)
    Note over Market,Grupo12: Phase 2: Mercado solicita acreditación del ítem a Grupo 12
    Market->>BusKafka: inventory.items.commands: ITEM_PROVISION_REQUESTED {orderId, holdId, studentId, courseId, itemPayload}
    
    alt Item Provisioning Failed
        Grupo12-->>BusKafka: inventory.items.events: ITEM_PROVISION_FAILED {orderId, holdId, reason}
        Market->>BusKafka: bank.holds.commands: HOLD_RELEASE_REQUESTED {holdId: "hld-99201"}
        Bank->>Bank: Releases locked coins without charge
        Market->>Market: Orden -> CANCELADA
        Market-->>Student: SSE: FAILED (ITEM_PROVISION_FAILED)
    else Item Provisioned OK
        Grupo12->>Grupo12: Acredita el ítem en su propio inventario (state: "AVAILABLE")
        Grupo12->>BusKafka: inventory.items.events: ITEM_PROVISIONED {orderId, holdId, inventoryItemId: "inv-8812", itemType, state}
    end
    end

    %% Phase 3: Final Debit
    rect rgb(254, 243, 199)
    Note over Market,Bank: Phase 3: Confirm Bank Hold (débito final, recién si la acreditación se corroboró)
    Market->>BusKafka: bank.holds.commands: HOLD_CONFIRM_REQUESTED {holdId: "hld-99201"}
    Bank->>Bank: Converts hold into final ledger debit
    Bank->>BusKafka: bank.holds.events: HOLD_CONFIRMED {holdId: "hld-99201", status: "COMMITTED"}
    
    Market->>Market: Confirms PurchaseOrder (status: CONFIRMED)
    Market-->>Student: SSE Push: "Purchase confirmed! Item added to your inventory."
    Market->>BusKafka: market.orders.events: PURCHASE_CONFIRMED
    end
```

---

## 5. Sequence Diagram: Auction Closing & Asset Settlement

```mermaid
sequenceDiagram
    autonumber
    participant Scheduler as Cron Scheduler (Market)
    participant Market as Market (Team 09)
    participant BusKafka as Kafka Bus
    participant Bank as Bank (Team 08)
    participant Grupo12 as Grupo 12 (Item Provisioning)

    Scheduler->>Market: Triggers auction expiry evaluation
    Market->>Market: Optimistic lock: OPEN -> CLOSING_IN_PROGRESS (EVALUATING_WINNER)

    alt Registered Bids Exist
        Market->>Market: Identifies winning bid (highest amount)

        %% Phase 1: Deliver Asset to Winner via Grupo 12 — happens BEFORE ledger confirmation
        Market->>Market: EVALUATING_WINNER -> CREDITING_ITEM
        Market->>BusKafka: inventory.items.commands: ITEM_PROVISION_REQUESTED {auctionId, winnerStudentId, courseId, itemPayload}
        alt Item Provisioning Failed
            Grupo12-->>BusKafka: inventory.items.events: ITEM_PROVISION_FAILED {auctionId, winnerStudentId, reason}
            Market->>Market: CREDITING_ITEM -> FAILED_SETTLEMENT (AWAITING_MANUAL_OR_CRON_RETRY)
            Note over Market: El hold del ganador todavía no se confirmó — nada que compensar, se reintenta la acreditación
        else Item Provisioned OK
            Grupo12->>BusKafka: inventory.items.events: ITEM_PROVISIONED {auctionId, winnerStudentId, inventoryItemId: "inv-9921", itemType, state}
            Market->>Market: CREDITING_ITEM -> CONFIRMING_LEDGER

            %% Phase 2: Confirm Winner Hold & Release Losers
            par Bank Settlement
                Market->>BusKafka: bank.holds.commands: HOLD_CONFIRM_REQUESTED {holdId: winningHoldId}
                Bank->>Bank: Commits ledger deduction
                Bank->>BusKafka: bank.holds.events: HOLD_CONFIRMED {holdId: winningHoldId}
            and Losers Hold Release
                Market->>BusKafka: bank.holds.commands: HOLD_RELEASE_REQUESTED {holdId: losingHoldId}
                Bank->>Bank: Releases funds without penalty
                Bank->>BusKafka: bank.holds.events: HOLD_RELEASED {holdId: losingHoldId}
            end

            Market->>Market: CONFIRMING_LEDGER -> RELEASING_LOSERS -> CLOSED
            Market->>BusKafka: market.auctions.events: AUCTION_CLOSED {auctionId, holdIds}
            Market->>BusKafka: market.auctions.events: AUCTION_AWARDED {auctionId, winnerStudentId}
        end
    else No Bids Registered
        Market->>Market: CLOSING_IN_PROGRESS -> MARKED_DESERTED -> CLOSED
    end
```
