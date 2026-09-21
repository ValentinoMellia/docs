# Contratos de comunicación — Mercado (Tema 09 / Grupo 11) para Sprint 1

> **Alcance: solo Catálogo (M-01) y Compra Directa (M-02)**, el trabajo activo en Sprint 1. Subastas (Fase 3, épica #577) no está en este sprint — su contrato de comunicación vigente sigue siendo [`Mercado/Subastas/03-contratos-eventos-e-idempotencia.md`](Mercado/Subastas/03-contratos-eventos-e-idempotencia.md); este documento no lo duplica.
>
> **Fuente de verdad:** todo lo que sigue está tomado o derivado de [`CONTEXTO-MERCADO-SPRINT1.md`](CONTEXTO-MERCADO-SPRINT1.md) §7, §9 y §10 (documento ya cerrado y auditado el 18/09/2026). Donde CONTEXTO no fija un path REST literal (endpoints de catálogo), se propone uno consistente con la convención ya usada en el resto de la plataforma (`/api/v1/<tema>/...`) — está marcado explícitamente como **propuesto**, a confirmar en diseño técnico.

---

## 1. Qué expone Mercado

### 1.1 Catálogo (M-01) — *propuesto, no fijado literalmente en CONTEXTO*

| Endpoint | Rol | Épica/HU | Propósito |
|---|---|---|---|
| `POST /api/v1/market/offers` | `ROLE_PROFESSOR` | HU-01.1 | Publicar una oferta de catálogo eligiendo un `ItemTemplate` y configurando sus parámetros (precio dentro del rango, magnitud, cargas). |
| `PATCH /api/v1/market/offers/{offerId}` | `ROLE_PROFESSOR` | HU-01.1 | Modificar una oferta ya publicada de su propio curso-cohorte. |
| `PATCH /api/v1/market/offers/{offerId}/status` | `ROLE_PROFESSOR` | HU-01.1 | Activar/desactivar una oferta. |
| `GET /api/v1/market/courses/{courseId}/offers` | `ROLE_STUDENT`, `ROLE_PROFESSOR` | HU-01.2 | Listado de ofertas activas del curso-cohorte (vitrina). |
| `GET /api/v1/market/offers/{offerId}` | `ROLE_STUDENT`, `ROLE_PROFESSOR` | HU-01.2 | Detalle de una oferta puntual. |
| `GET /api/v1/market/templates` | `ROLE_PROFESSOR` | HU-01.1 | Listado de `ItemTemplate` base disponibles para armar una oferta. |

Todos requieren las cabeceras inyectadas por el Gateway (§1.3) y devuelven `403 {ROLE_NOT_PERMITTED}` si el rol no corresponde, o si el curso-cohorte de la oferta no coincide con el del usuario (curaduría aislada por curso, HU-01.3).

### 1.2 Compra directa (M-02) — tomado de CONTEXTO §7 / `flujo-comunicacion-banco.md`

| Endpoint | Rol | Propósito |
|---|---|---|
| `POST /api/v1/market/orders` | `ROLE_STUDENT` | Inicia una orden de compra. Body: `{offerId, courseId, idempotencyKey}`. Responde `202 Accepted {orderId, status:"PROCESSING", sseStreamUrl}` — nunca bloquea esperando a Banco. |
| `GET /api/v1/market/orders/stream/{orderId}` | `ROLE_STUDENT` | Canal SSE unidireccional con el progreso de la orden (hold → acreditación → confirmación). |
| `GET /api/v1/market/admin/metrics?courseId=` | `ROLE_PROFESSOR` | Snapshot agregado de métricas de Mercado, frescura máxima 15 min (§9.5). |

### 1.3 Cabeceras que Mercado recibe de todo request entrante (Gateway, §9.3)

`X-User-Id`, `X-Roles`, `X-User-Email`, `X-Forwarded-For`. Mercado nunca valida JWT — confía en el Gateway.

---

## 2. Qué emite Mercado (eventos, CONTEXTO §10)

Convención obligatoria (decisión #4): `MAYUSCULAS_SNAKE_CASE`, en inglés, envoltura estándar de 6 campos (`EventEnvelope<T>` con `eventId`, `eventType`, `eventVersion`, `timestamp`, `producer`, `payload`).

| Evento | Tópico | Cuándo | Consumidores conocidos |
|---|---|---|---|
| `CATALOG_OFFER_PUBLISHED` | `market.catalog.events` | El profesor publica una nueva oferta (HU-01.1) | Notificaciones (11) |
| `PURCHASE_CONFIRMED` | `market.orders.events` | Compra directa confirmada (fin feliz de M-02) | Notificaciones (11) |

Eliminado explícitamente por decisión #13: `ITEM_CONSUMED` — Mercado ya no lo publica (el consumo de ítems es contrato exclusivo Motor de Desafíos ↔ Grupo 12).

---

## 3. Qué necesita Mercado de otros (dependencias, CONTEXTO §9)

### 3.1 Banco / Grupo 12 — dependencia crítica bloqueante (§9.1, §7)

Saga de 3 pasos, síncrono vía Gateway para comandos con respuesta inmediata, asíncrono vía Kafka para hechos consumados. Envoltura estándar de 6 campos (`EventEnvelope<T>` con `eventVersion`). `partitionKey = studentId` para compra directa.

```
1. HOLD_CREATE_REQUESTED  {orderId, studentId, courseId, amount, currency, orderType:"DIRECT_PURCHASE"}
   → HOLD_CREATED {holdId, orderId, studentId, courseId, amount, status:"PENDING", expiresAt}
   → HOLD_REJECTED {orderId, studentId, reason} (ej. INSUFFICIENT_FUNDS)

2. ITEM_PROVISION_REQUESTED {orderId, holdId, studentId, courseId, itemPayload}
   → ITEM_PROVISIONED {orderId, holdId, studentId, courseId, inventoryItemId, itemType, state}
   → ITEM_PROVISION_FAILED {orderId, holdId, reason}
      → Mercado responde con HOLD_RELEASE_REQUESTED {holdId, reason} → HOLD_RELEASED

3. HOLD_CONFIRM_REQUESTED {holdId, orderId, studentId, courseId, amount}
   → HOLD_CONFIRMED {holdId, orderId, studentId, amountDebited, ledgerEntryId, status:"COMMITTED"}
```

Tópicos: `bank.holds.commands` (Mercado→Banco), `bank.holds.events` (Banco→Mercado), `inventory.items.commands` (Mercado→Grupo 12), `inventory.items.events` (Grupo 12→Mercado).

**Regla de orden (decisión #8, no negociable):** nunca confirmar el débito (`HOLD_CONFIRM_REQUESTED`) antes de que la acreditación del ítem se haya corroborado (`ITEM_PROVISIONED`). Este orden es el que sostiene que "pagó pero no recibió el ítem" no sea un escenario normal a compensar.

**Pendiente de acuerdo formal con Grupo 12 (no bloquea Sprint 1, CONTEXTO §7 y §9.1):** la posible simplificación de colapsar hold+acreditación+confirmación en una sola saga interna de Grupo 12; códigos de error diferenciados completos; parámetros exactos de reintento/timeout.

### 3.2 Cursos y Matrícula (Tema 02, §9.2)

- **Síncrono:** `GET /api/v1/courses/{courseId}/students/{studentId}/enrollment-status` → `{courseId, studentId, enrolled, status, enrolledAt}`. Si `enrolled:false` → `403 {NOT_ENROLLED_IN_COURSE}`. **Este es el contrato clave para US-1036** ("Comprar solo si estoy cursando la materia").
- **Asíncrono**, tópico `courses.lifecycle`:
  - `COURSE_ARCHIVED` (partitionKey `courseId`) → Mercado bloquea nuevas órdenes/compras del curso.
  - `STUDENT_UNENROLLED` (partitionKey `studentId`) → mismo tratamiento que `COURSE_ARCHIVED`, solo para ese alumno (decisión #10).

### 3.3 Backoffice (Tema 12, §9.5)

- **Asíncrono:** tópico `backoffice.parametros`, evento `PARAMETRO_ACTUALIZADO` → Mercado invalida caché local de PAR-06/PAR-07, hot-reload sin downtime.
- **Síncrono (decisión #9, nuevo):** `GET /api/v1/backoffice/parametros/{id}` (rol `PROFESSOR`/`ADMIN`) — para pedir el precio vigente al instante cuando la frescura del caché no alcanza (ej. al mostrar catálogo o validar una orden).

### 3.4 Notificaciones (Tema 11, §9.6)

Solo consumidor — no expone nada que Mercado necesite llamar. Consume `CATALOG_OFFER_PUBLISHED` y `PURCHASE_CONFIRMED` de la Sección 2.

---

## 4. Matriz cruzada — tasks de Valentino (US-138, US-139, US-142)

| Task | US padre | Punto de integración de este documento |
|---|---|---|
| #976 T01 Modelar orden de compra | US-138 | §1.2 (contrato de `Orden`, sin sección propia de endpoint) |
| #977 T02 Endpoint de compra | US-138 | §1.2 — `POST /api/v1/market/orders` + `GET .../stream/{orderId}` |
| #978 T03 Reserva de monedas a Banco | US-138 | §3.1 — paso 1 de la saga (`HOLD_CREATE_REQUESTED`/`HOLD_CREATED`/`HOLD_REJECTED`) |
| #979 T04 Acreditación del ítem | US-138 | §3.1 — paso 2 de la saga (`ITEM_PROVISION_REQUESTED`/`PROVISIONED`/`FAILED`) |
| #980 T05 Confirmar cobro y cerrar orden | US-138 | §3.1 — paso 3 de la saga (`HOLD_CONFIRM_REQUESTED`/`HOLD_CONFIRMED`) + §2 (`PURCHASE_CONFIRMED`) |
| #981 T06 Pruebas (camino feliz/saldo/oferta inactiva) | US-138 | §3.1 completo (para simular `HOLD_REJECTED`/`ITEM_PROVISION_FAILED`) |
| #986 T01 Clave de idempotencia en contrato | US-139 | §1.2 (`idempotencyKey` en `POST /orders`) + §3.1 (`commandId` hacia Banco) |
| #987 T02 Índice único + huella | US-139 | Interno a Mercado (BD propia `Orden.idempotencyKey`), sin contrato externo |
| #988 T03 Deshabilitar botón durante operación | US-139 | §1.2 (`GET .../stream/{orderId}` para feedback de estado) |
| #989 T04 Pruebas de doble envío/conflicto | US-139 | §1.2 + §3.1 (verificar que un reintento con la misma `idempotencyKey` no dispare una segunda saga) |
| #1002 T01 Propagar motivo de rechazo por tope | US-142 | §3.2 (validación de tope de vidas es una regla de negocio previa al hold, no de Banco directamente — ver CONTEXTO §9.7) |
| #1003 T02 Mensajes distintos tope vs saldo | US-142 | §3.1 (`HOLD_REJECTED.reason`) vs. validación interna de tope (PAR-12, Tema 10) |
| #1004 T03 Pruebas de compra bajo/en el tope | US-142 | §3.1 + validación de tope interna |

---

## 5. Referencias

- [`CONTEXTO-MERCADO-SPRINT1.md`](CONTEXTO-MERCADO-SPRINT1.md) — fuente de verdad de todo lo anterior (§7, §9, §10).
- [`Comunicacion/Grupo-08-Banco/flujo-mercado-inventario.md`](Comunicacion/Grupo-08-Banco/flujo-mercado-inventario.md) — especificación consolidada y ampliada de la saga de compra directa (Mercado ↔ Banco ↔ Inventario) con diagramas, idempotencia y análisis de ambigüedades.
- [`Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md`](Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md) — documento base de integración con Banco (vigente salvo modelo de stock y posesión de inventario).
- [`Mercado/Subastas/03-contratos-eventos-e-idempotencia.md`](Mercado/Subastas/03-contratos-eventos-e-idempotencia.md) — contrato equivalente para subastas (Fase 3, fuera de este sprint).
