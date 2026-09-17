# Architectural Diagrams — Market (Team 09)
## Distributed Gamified Platform · Aula Quest (TUP UTN FRC)

---

## 1. Context Diagram: Market & External Microservices

Market acts as an orchestrating Kiosk / Storefront. In accordance with platform design, the Two-Phase Hold Saga, and the **Local Stock Hold (Fail-Fast)** pattern:
- **Market (Team 09):** Holds local product stock (`StockHold`) before touching external services, orchestrates the purchase saga, and curates the Open Catalog.
- **Bank (Team 08):** Manages coin balances, handles `BalanceHold` commands, and settles ledger debits.
- **Inventory Service:** Separate microservice managing the student backpack (`student_inventory`), active slots, charges, and runtime effect execution.

```mermaid
graph TB
    subgraph Web Clients
        Student[Student Frontend - Storefront / Auctions]
        Professor[Professor Frontend - Catalog Curation / Auctions]
    end

    GW[API Gateway / Perimetral Security]
    Student --> GW
    Professor --> GW

    GW -->|HTTP REST| T09[Team 09 · Market<br/><b>Kiosk / Open Catalog / Stock Hold / Auctions</b>]

    subgraph Kafka Event Bus Ecosystem
        T09 -->|bank.holds.commands| BUS[(Apache Kafka<br/>Event Bus)]
        BUS -->|bank.holds.events| T09
        
        T09 -->|inventory.items.commands| BUS
        BUS -->|inventory.items.events| T09

        BUS <-->|Balance Holds & Ledger Debits| T08[Team 08 · Bank<br/><b>Ledger, Coin Holds & Balances</b>]
        BUS <-->|Item Persistence & Charges| T_INV[Inventory Service<br/><b>Student Backpack & Effects</b>]
    end

    BUS -.->|Informative Alerts| T11[Team 11 · Notifications]
```

---

## 2. Market Domain Model (Open Catalog, Stock Holds & Two-Layer Architecture)

Market manages templates, cohort offerings, purchase orders, **local stock holds**, and auctions:

```mermaid
classDiagram
    class ItemBaseTemplate {
        +UUID id
        +ItemType type  // SHIELD | BOOST_XP | BOOST_COINS | LIFE
        +String defaultName
        +String defaultDescription
        +String iconUrl
        +boolean active
    }

    class CourseCatalogOffer {
        +UUID id
        +UUID courseCohortId
        +UUID itemBaseTemplateId
        +String customName
        +String customDescription
        +int coinPrice
        +Integer stock  // null = unlimited, > 0 = finite cohort pool
        +Integer availableStock  // remaining units not committed or held
        +boolean active
        +ItemConfiguration configuration
        +Instant createdAt
        +Instant updatedAt
    }

    class StockHold {
        +UUID id
        +UUID catalogOfferId
        +UUID purchaseOrderId
        +UUID studentId
        +StockHoldStatus status  // PENDING | COMMITTED | RELEASED | EXPIRED
        +Instant expiresAt  // default: 5 minutes TTL
        +Instant createdAt
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
        +UUID stockHoldId  // Local Market stock lock
        +UUID bankHoldId   // Bank coin balance lock
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

    ItemBaseTemplate "1" --> "0..*" CourseCatalogOffer : instantiates
    CourseCatalogOffer "1" *-- "1" ShieldConfiguration : when type == SHIELD
    CourseCatalogOffer "1" *-- "1" BoostConfiguration : when type == BOOST_*
    CourseCatalogOffer "1" *-- "1" LifeConfiguration : when type == LIFE

    CourseCatalogOffer "1" --> "0..*" StockHold : locks_stock
    CourseCatalogOffer "1" --> "0..*" PurchaseOrder : transactions
    PurchaseOrder "1" o-- "0..1" StockHold : references
    CourseCatalogOffer "1" --> "0..*" Auction : featured_in
    Auction "1" --> "0..*" Bid : receives
```

---

## 3. State Machines

### 3.1 Local Stock Hold State Machine (Market)
```mermaid
stateDiagram-v2
    [*] --> PENDING : Atomic reservation (availableStock - 1, TTL: 5m)
    PENDING --> COMMITTED : Saga succeeded (HOLD_CONFIRMED in Bank)
    PENDING --> RELEASED : Bank rejected funds or Inventory failed (availableStock + 1)
    PENDING --> EXPIRED : TTL expired without resolution (reconciliation scheduler)
    COMMITTED --> [*]
    RELEASED --> [*]
    EXPIRED --> [*]
```

### 3.2 Purchase Order State Machine (Two-Phase Hold Saga)
```mermaid
stateDiagram-v2
    [*] --> CREATED : Student triggers purchase via REST
    CREATED --> STOCK_HELD : Local StockHold created (if finite stock)
    CREATED --> OUT_OF_STOCK : No available units (HTTP 409 Conflict)
    
    STOCK_HELD --> HOLD_REQUESTED : Emits HOLD_CREATE_REQUESTED to Bank
    HOLD_REQUESTED --> PROVISIONING_ITEM : Receives HOLD_CREATED from Bank
    HOLD_REQUESTED --> REJECTED_FUNDS : Receives HOLD_REJECTED -> Releases StockHold
    
    PROVISIONING_ITEM --> DEBIT_REQUESTED : Receives ITEM_PROVISIONED from Inventory
    PROVISIONING_ITEM --> COMPENSATING : Receives ITEM_PROVISION_FAILED from Inventory
    
    COMPENSATING --> COMPENSATED_RELEASED : Releases Bank Hold & Releases StockHold
    
    DEBIT_REQUESTED --> CONFIRMED : Receives HOLD_CONFIRMED -> Commits StockHold
    
    CONFIRMED --> [*]
    OUT_OF_STOCK --> [*]
    REJECTED_FUNDS --> [*]
    COMPENSATED_RELEASED --> [*]
```

---

## 4. Sequence Diagram: Direct Purchase with Dual Holds (Stock Hold + Bank Hold)

```mermaid
sequenceDiagram
    autonumber
    actor Student as Student (Web Frontend)
    participant GW as API Gateway
    participant Market as Market (Team 09)
    participant BusKafka as Kafka Bus
    participant Bank as Bank (Team 08)
    participant Inventory as Inventory Service

    Student->>GW: POST /api/v1/market/orders {offerId: "item-course-9912", courseId: "COURSE_PROG4_2026"}
    GW->>Market: Forwards request with auth headers (X-User-Id, X-Roles)

    %% Step 0: Local Stock Hold (Fail-Fast)
    rect rgb(254, 242, 242)
    Note over Market: Phase 0: Local Stock Hold (Fail-Fast in 2ms)
    alt Finite Stock Configured & Available
        Market->>Market: Atomic UPDATE: availableStock - 1<br/>Creates StockHold (status: PENDING, ttl: 5m)
        Market-->>Student: 202 Accepted {orderId: "ord-88391a", status: "PROCESSING"}
    else Stock Depleted (availableStock == 0)
        Market-->>Student: 409 Conflict {error: "OUT_OF_STOCK", message: "Item is sold out"}
    end
    end

    %% Phase 1: Coin Hold in Bank
    rect rgb(245, 243, 255)
    Note over Market,Bank: Phase 1: Reserve Coins in Bank (BalanceHold)
    Market->>BusKafka: bank.holds.commands: HOLD_CREATE_REQUESTED {orderId, studentId, amount: 350}
    
    alt Insufficient Balance in Bank
        Bank-->>BusKafka: bank.holds.events: HOLD_REJECTED {orderId, reason: "INSUFFICIENT_FUNDS"}
        Market->>Market: Releases StockHold (status: RELEASED, availableStock + 1)
        Market-->>Student: SSE: FAILED (INSUFFICIENT_FUNDS)
    else Sufficient Balance in Bank
        Bank->>Bank: Locks 350 coins (status: PENDING, holdId: "hld-99201")
        Bank->>BusKafka: bank.holds.events: HOLD_CREATED {holdId: "hld-99201", status: "PENDING"}
    end
    end

    %% Phase 2: Provision Item to Inventory
    rect rgb(236, 253, 245)
    Note over Market,Inventory: Phase 2: Deliver Item into Student Inventory
    Market->>BusKafka: inventory.items.commands: ITEM_PROVISION_REQUESTED {orderId, holdId, itemPayload}
    
    alt Inventory Persistence Error
        Inventory-->>BusKafka: inventory.items.events: ITEM_PROVISION_FAILED {orderId}
        Market->>BusKafka: bank.holds.commands: HOLD_RELEASE_REQUESTED {holdId: "hld-99201"}
        Bank->>Bank: Releases locked coins without charge
        Market->>Market: Releases StockHold (status: RELEASED, availableStock + 1)
        Market-->>Student: SSE: FAILED (DELIVERY_FAILED)
    else Inventory Persisted OK
        Inventory->>Inventory: Persists item in student_inventory (state: "AVAILABLE")
        Inventory->>BusKafka: inventory.items.events: ITEM_PROVISIONED {orderId, inventoryItemId: "inv-8812"}
    end
    end

    %% Phase 3: Final Debit & Stock Commit
    rect rgb(254, 243, 199)
    Note over Market,Bank: Phase 3: Confirm Bank Hold & Commit Stock
    Market->>BusKafka: bank.holds.commands: HOLD_CONFIRM_REQUESTED {holdId: "hld-99201"}
    Bank->>Bank: Converts hold into final ledger debit
    Bank->>BusKafka: bank.holds.events: HOLD_CONFIRMED {holdId: "hld-99201", status: "COMMITTED"}
    
    Market->>Market: Commits StockHold (status: COMMITTED)
    Market->>Market: Confirms PurchaseOrder (status: CONFIRMED)
    Market-->>Student: SSE Push: "Purchase confirmed! Item added to your inventory."
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
    participant Inventory as Inventory Service

    Scheduler->>Market: Triggers auction expiry evaluation
    Market->>Market: Optimistic lock: OPEN -> CLOSING

    alt Registered Bids Exist
        Market->>Market: Identifies winning bid (highest amount)
        
        %% Phase 1: Deliver Asset to Winner
        Market->>BusKafka: inventory.items.commands: ITEM_PROVISION_REQUESTED {studentId: winnerId, itemPayload}
        Inventory->>BusKafka: inventory.items.events: ITEM_PROVISIONED {inventoryItemId: "inv-9921"}

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

        Market->>Market: Updates Auction -> ADJUDICATED
    else No Bids Registered
        Market->>Market: Updates Auction -> DESERTED
    end
```
