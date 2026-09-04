# Diagramas base — Tema 09 Mercado

Borradores en Mermaid para discutir y corregir en equipo. **No los tomen como
verdad**: la mitad del valor de hoy está en romperlos y rehacerlos juntos.

Se renderizan en GitHub, GitLab, Notion, Obsidian, VS Code (extensión Mermaid) y en
mermaid.live.

---

## 1. Contexto: el Mercado y sus vecinos

```mermaid
graph TB
    subgraph Cliente
        FE[Front End]
    end
    GW[API Gateway]
    FE --> GW

    GW --> T09[Tema 09 · Mercado<br/>catálogo, órdenes, inventario, subastas]
    T09 -. sincrónico vía gateway .-> GW

    GW --> T08[Tema 08 · Banco<br/>ledger, saldos, reservas]
    GW --> T10[Tema 10 · Roadmap<br/>XP, vidas, insignias]
    GW --> T02[Tema 02 · Cursos<br/>cohorte, matrícula]
    GW --> T12[Tema 12 · Backoffice<br/>parámetros PAR]

    T09 -->|publica| BUS[(Bus de eventos<br/>contrato: Tema 11)]
    T02 -->|curso.archivado| BUS
    BUS --> T11[Tema 11 · Notificaciones]
    BUS --> T10
    BUS --> T12
```

**Regla que el diagrama tiene que respetar:** el Mercado nunca llama directo a otro
microservicio ni lee su base. Sale y vuelve a entrar por el gateway.

---

## 2. Modelo de dominio (borrador)

```mermaid
classDiagram
    class ItemDefinicion {
        +UUID id
        +String nombre
        +TipoItem tipo  // VIDA | EQUIPAMIENTO
        +String efecto  // contrato con Tema 10
        +boolean consumible
        +boolean activo
    }

    class OfertaCatalogo {
        +UUID id
        +UUID cursoCohorteId
        +UUID itemDefinicionId
        +int precioMonedas
        +Integer stock  // null = ilimitado
        +EstadoOferta estado
    }

    class Orden {
        +UUID id
        +UUID cursoCohorteId
        +UUID alumnoId
        +UUID ofertaId
        +int precioAplicado  // snapshot RF-CFG-06
        +UUID reservaId
        +String idempotencyKey
        +EstadoOrden estado
        +Instant creadaEn
    }

    class Subasta {
        +UUID id
        +UUID cursoCohorteId
        +UUID itemDefinicionId
        +UUID profesorId
        +Instant inicio
        +Instant fin
        +Integer pujaMinima
        +EstadoSubasta estado
        +long version
    }

    class Puja {
        +UUID id
        +UUID subastaId
        +UUID alumnoId
        +int monto
        +UUID reservaId
        +EstadoPuja estado
        +Instant creadaEn
    }

    class ItemInventario {
        +UUID id
        +UUID cursoCohorteId
        +UUID alumnoId
        +UUID itemDefinicionId
        +OrigenItem origen  // COMPRA | SUBASTA | DESAFIO
        +EstadoItem estado
        +Instant consumidoEn
        +long version
    }

    ItemDefinicion "1" --> "0..*" OfertaCatalogo
    OfertaCatalogo "1" --> "0..*" Orden
    Orden "1" --> "0..1" ItemInventario : entrega
    ItemDefinicion "1" --> "0..*" Subasta
    Subasta "1" --> "0..*" Puja
    Subasta "1" --> "0..1" ItemInventario : adjudica
```

Preguntas para la discusión:
- ¿La vida entra al inventario o va directo al Tema 10 sin instancia local?
- ¿`OfertaCatalogo` guarda el precio o lo lee siempre del Tema 12? (Sugerencia: lo lee,
  pero la `Orden` guarda el snapshot.)
- ¿Hace falta `stock`? El PRD no lo pide para compra directa.

---

## 3. Máquina de estados — Orden de compra directa

```mermaid
stateDiagram-v2
    [*] --> CREADA
    CREADA --> RESERVA_SOLICITADA : solicitar reserva al Banco
    RESERVA_SOLICITADA --> RESERVADA : reserva OK
    RESERVA_SOLICITADA --> RECHAZADA_SALDO : saldo insuficiente
    RESERVA_SOLICITADA --> FALLIDA_BANCO : timeout / error
    RESERVADA --> CONFIRMADA : ítem entregado + confirmar(reservaId)
    RESERVADA --> CANCELADA : falla la entrega -> liberar(reservaId)
    RESERVADA --> EXPIRADA : TTL vencido -> liberar
    CONFIRMADA --> [*]
    RECHAZADA_SALDO --> [*]
    FALLIDA_BANCO --> [*]
    CANCELADA --> [*]
    EXPIRADA --> [*]
```

---

## 4. Máquina de estados — Subasta (RF-INT-05, RF-INT-06)

```mermaid
stateDiagram-v2
    [*] --> BORRADOR
    BORRADOR --> PROGRAMADA : profesor define duración y puja mínima
    PROGRAMADA --> ABIERTA : llega la fecha de inicio
    ABIERTA --> EN_CIERRE : llega la fecha de fin
    ABIERTA --> CANCELADA : profesor cancela (libera todas las pujas)
    EN_CIERRE --> ADJUDICADA : hay pujas -> confirma la ganadora, libera el resto
    EN_CIERRE --> DESIERTA : sin pujas -> asset sin asignar
    ADJUDICADA --> [*]
    DESIERTA --> [*]
    CANCELADA --> [*]
```

`EN_CIERRE` no está en el PRD: lo agregamos porque el cierre no es instantáneo (hay que
confirmar una reserva y liberar N). Sin ese estado intermedio, dos ejecuciones
concurrentes del cierre pueden adjudicar dos veces.

---

## 5. Máquina de estados — Puja

```mermaid
stateDiagram-v2
    [*] --> ACTIVA : reserva de monedas OK
    ACTIVA --> SUPERADA : el alumno aumenta su oferta (nueva puja)
    ACTIVA --> GANADORA : cierre, es la mayor
    ACTIVA --> LIBERADA : cierre sin ganar / cancelación
    SUPERADA --> [*]
    GANADORA --> [*]
    LIBERADA --> [*]
```

Decisión pendiente con el Tema 08: cuando el alumno **aumenta** su oferta, ¿se reserva
solo el delta (más eficiente, más difícil de razonar) o se libera la anterior y se
reserva el total (más simple, con una ventana en la que el alumno podría gastar esas
monedas en otro lado)?

---

## 6. Máquina de estados — Ítem de inventario (RF-REC-05)

```mermaid
stateDiagram-v2
    [*] --> DISPONIBLE : alta por compra / subasta / desafío
    DISPONIBLE --> CONSUMIDO : consumo idempotente (uso único)
    DISPONIBLE --> EXPIRADO : vencimiento de ítem (extra)
    DISPONIBLE --> INACTIVO : curso archivado / desmatriculación (a definir)
    CONSUMIDO --> [*]
    EXPIRADO --> [*]
    INACTIVO --> [*]
```

---

## 7. Secuencia — Compra directa (camino feliz)

```mermaid
sequenceDiagram
    actor A as Alumno
    participant GW as API Gateway
    participant M as Mercado (09)
    participant B as Banco (08)
    participant BUS as Bus de eventos
    participant R as Roadmap (10)

    A->>GW: POST /mercado/ordenes {ofertaId, idempotencyKey}
    GW->>M: ruteo con token validado
    M->>M: validar cohorte, oferta activa, precio vigente (PAR)
    M->>GW: POST /banco/reservas {alumno, cohorte, monto, key}
    GW->>B: ...
    B-->>M: 201 {reservaId}
    M->>M: crear ItemInventario / preparar entrega
    M->>GW: POST /banco/reservas/{id}/confirmar
    GW->>B: ...
    B-->>M: 200 OK (ledger debitado)
    M->>M: Orden = CONFIRMADA (+ outbox)
    M-->>A: 201 orden confirmada
    M->>BUS: mercado.compra.confirmada
    BUS->>R: acreditar vida / registrar equipamiento
```

### Variantes que hay que dibujar también

- **Saldo insuficiente**: el Banco rechaza en el paso de reserva → orden
  `RECHAZADA_SALDO`, nada que compensar.
- **Falla la entrega después de reservar**: `liberar(reservaId)` → orden `CANCELADA`.
- **Timeout al confirmar**: reintento con la misma clave de idempotencia; si persiste,
  la reserva expira por TTL del lado del Banco y el job de reconciliación cierra la
  orden.
- **El Roadmap rechaza la vida (máximo alcanzado)**: por eso conviene validar **antes**
  de reservar, o definir una compensación explícita (devolver monedas + notificar).

---

## 8. Secuencia — Cierre de subasta

```mermaid
sequenceDiagram
    participant S as Scheduler (Mercado)
    participant M as Mercado (09)
    participant GW as API Gateway
    participant B as Banco (08)
    participant BUS as Bus de eventos

    S->>M: cerrar subastas con fin <= ahora
    M->>M: lock optimista: ABIERTA -> EN_CIERRE
    alt hay pujas
        M->>M: determinar puja ganadora (mayor monto, desempate por fecha)
        M->>GW: confirmar(reservaId de la ganadora)
        GW->>B: ...
        B-->>M: OK
        loop por cada puja perdedora
            M->>GW: liberar(reservaId)
            GW->>B: ...
        end
        M->>M: crear ItemInventario para el ganador; estado ADJUDICADA
        M->>BUS: mercado.subasta.cerrada {ganador}
    else sin pujas
        M->>M: estado DESIERTA
        M->>BUS: mercado.subasta.cerrada {desierta}
    end
```

Puntos de diseño a resolver: idempotencia del cierre (si el scheduler corre en dos
instancias), criterio de desempate ante montos iguales, y qué pasa si la confirmación
de la ganadora falla (¿se adjudica al segundo?).

---

## 9. Ciclo de vida de la cohorte visto desde el Mercado

```mermaid
stateDiagram-v2
    direction LR
    Creacion --> Configuracion
    Configuracion --> Activacion : requiere calibración aprobada (Tema 07)
    Activacion --> Dictado
    Dictado --> Cierre
    Cierre --> Archivado

    note right of Configuracion
        Mercado: se puede armar el catálogo
        de la cohorte, sin operaciones de alumnos
    end note
    note right of Dictado
        Mercado: compras, subastas, consumo
    end note
    note right of Archivado
        Mercado: solo lectura (RF-CUR-09).
        Cerrar subastas abiertas y liberar reservas.
    end note
```
