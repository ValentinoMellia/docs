# Accounting (Banco): estado del repo y de sus contratos respecto de Mercado

> **Estado al 30/09/2026**, verificado sobre `2026-P4-BE/tpi-accounting`, rama **`develop`** (commit `3013f6c`). La rama `main` es un esqueleto (controlador de ping y `ErrorApi`, 350+ commits atrás): **no sirve para saber qué hay**. Entre el análisis del 29/09 (`f965420`) y hoy hubo 12 commits, todos de tests y fixtures; ninguno toca `src/main`, así que nada de lo que sigue cambió.
> Se leyó el código y la documentación propia del repo; la app no se ejecutó. Lo que no se pudo comprobar está marcado como **NO VERIFICADO**.
> "Accounting" es el servicio que los documentos anteriores llamaban Banco / Grupo 12 / Inventario: en el código contiene **cuentas y monedas (ledger), retenciones de monedas (holds), vidas, retenciones de vidas, inventario del alumno y liquidación de desafíos**. No existe un repo de Inventario aparte.
> Complementa a [`estado-integracion-mercado-accounting.md`](./estado-integracion-mercado-accounting.md) (desajustes y flujo) y a [`contrato-integracion-mercado-accounting.md`](./contrato-integracion-mercado-accounting.md) (contrato de comando único, propuesto).

## 1. Resumen para Mercado

| Lo que Mercado necesita | Estado en Accounting hoy |
|---|---|
| Retener monedas de un alumno (hold) | ⚠️ **Implementado y testeado, pero apagado y sin poder encenderse.** Solo por Kafka, sin REST |
| Confirmar / liberar / aumentar un hold | ⚠️ Ídem (mismos comandos) |
| Acreditar un ítem comprado | ⚠️ Implementado (`ITEM_CONFIRMED`) pero con un catálogo **placeholder**: solo 3 ids de prueba |
| Saber si un ítem falló | ❌ No hay evento de error; va a DLT sin aviso |
| Consultar un hold o el saldo (servicios) | ❌ No existe endpoint para `MS` |
| Comprar vidas | ❌ Solo en una rama sin PR (`feature/lives-purchase-credit`) |
| Expiración automática de holds | ❌ No existe; no publica `HOLD_EXPIRED` |
| Vencimiento/duración de un ítem | ❌ El modelo no tiene esos campos |
| Soporte para subastas | ⚠️ Un hold por postor que puede subir; sin liberación en lote ni ranking |
| Liquidar una compra en un solo comando | ❌ No existe (`PURCHASE_SETTLEMENT_*` no está en el código) |

**Conclusión:** Accounting tiene construido el motor de monedas, ledger e inventario, pero **la puerta de entrada para Mercado (los hold commands) está cerrada** y la de ítems solo acepta datos de prueba. Mientras eso siga así, Mercado no puede completar una compra contra el servicio real.

## 2. El repo hoy

| Aspecto | Detalle |
|---|---|
| Stack | Spring Boot MVC, JPA/Hibernate, **MySQL 8 + Flyway** (esquema versionado), Spring Kafka, Eureka (`accounting-service`) |
| Ruta y puertos | prefijo `/api/accounting`, puerto `8090` (gestión `8091`) |
| Ramas | `develop` tiene todo; `main` es un esqueleto |
| PRs | Sin PRs abiertos; 60+ mergeados entre el 22 y el 29/09 |
| Issues abiertos | `#1` (setup), `#5` (DB setup) |
| Ramas sin mergear con código relevante | `feature/lives-purchase-credit` (+8 commits, **sin PR**: compra de vidas), `feature/life-holds-challenge-reservation` (+6, PR #44 cerrado) y `feature/life-holds-expiration` (+16, PR #45 cerrado): ambas de *life holds*, no de monedas; `backup` |
| Migraciones | `V1` cuentas, `V2` ledger de monedas, `V3` ledger de vidas, `V4` ítems de inventario, `V5` `coin_holds`, `V6` `life_holds`, `V7` `outbox_events`, `V8` `processed_events`, `V9`–`V12` auditoría y ajustes, más parámetros de vidas, jobs de archivado de cohortes e integridad del ledger de vidas |
| Consumidores de Kafka | **Apagados por defecto** (`accounting.messaging.consumers-enabled=${KAFKA_CONSUMERS_ENABLED:false}`); la plantilla de plataforma los enciende con `true` |

Módulos y quién los trabajó (según la cronología de PRs): mensajería/plataforma, holds de monedas (`#21`, `#31` comandos de Mercado), ledger de monedas (`#14`, `#52`, `#53`), inventario (`#15`, `#24`, `#25`, `#41`, `#42`, `#59`), cuentas (`#18`, `#33`, `#36`, `#57`), vidas y life holds, y liquidación de desafíos (`#55`, `#58`, `#60`, `#61`).

## 3. Contratos: qué documentos existen y cuál manda

| Fuente | Qué es | Vigencia |
|---|---|---|
| Skill `contratos-kafka` v3 (`.skill-hub/`) | Contrato de plataforma: topic único por equipo, envelope de 6 campos | **Manda.** Es lo que Accounting implementó |
| `docs/contracts/accounting-service.asyncapi.yaml` v1.1.0 | AsyncAPI del servicio | Vigente con errores: marca `receiveHoldCommands` como *implemented* aunque el flag está apagado y falta el resender; dice que `accounting.events` no existía en el broker al escribirse (**NO VERIFICADO** si sigue así) |
| `docs/app_doc/persona-4-entrega-c.md` | Descripción **autoritativa** de los 4 comandos de hold de Mercado, tabla de rechazos y pendientes | Vigente; algunas razones figuran como "a confirmar con Mercado" |
| `docs/accounting-modelo.md` | Modelo de dominio y propuestas (compras y subastas con Mercado, `GET /holds/{holdId}`, `HoldExpirationScheduler`, `releaseHoldsByOrder`) | **Mezcla lo construido con propuestas**; nombra el topic `market.orders.events` y los productores `tema-09-mercado` / `topic-09-marketplace`, que el código no usa |
| `docs/app_doc/persona-8-*.md`, `persona-3-entrega-a.md` | Entregas internas (topics, reenvío, compra de vidas) | Parcialmente superados: mencionan `accounting.holds.commands/events`, hoy reemplazados por `accounting.events` |

Regla práctica: donde el modelo o el AsyncAPI difieran del código, **manda el código**. La tabla de diferencias está en §9.

## 4. Transporte Kafka

| Aspecto | Valor (verificado en código) |
|---|---|
| Consume | Un único listener (`accounting-inbound`) sobre los topics con al menos un handler activo entre `accounting.events`, `market.events` y `challenges.events`; listeners aparte para `courses.events` y `administration.events` |
| Publica | **Solo `accounting.events`**, con clave `studentId:courseId`. Tipos de agregado: `COINS_LEDGER`, `LIVES_LEDGER`, `INVENTORY_ITEM`, `COIN_HOLD`, `LIFE_HOLD`, `CHALLENGE` |
| Grupo / productor | `tema-08-accounting-service-group` / `tema-08-accounting-service` |
| Formato | JSON de texto (sin cabeceras de tipo); `acks=all`, productor idempotente; consumo con ack por registro tras el commit |
| Envelope | 6 campos: `eventId` (UUID canónico), `eventType`, `eventVersion` (entero exactamente `1`), `timestamp` (ISO-8601), `producer`, `payload` (objeto) |
| Validación de entrada | Se lee primero `eventType` (tipos no registrados en ese topic se ignoran **y se confirma el offset**); luego `eventId` UUID, `eventVersion` = 1, `producer` no vacío, `payload` objeto. `timestamp` y cabeceras de correlación (`traceparent`, `X-Request-Id`) **no** se validan ni se publican |
| Fallos | 3 intentos con 2 s de espera y luego `<topic>.DLT`; un mensaje malformado va **directo** al DLT sin reintento |
| Salida | Outbox transaccional (`outbox_events`), relay cada 5 s (lotes de 100, un solo consumidor con lock, orden por clave, entrega al menos una vez con el mismo `eventId`); limpieza de filas publicadas tras 7 días. **Espere hasta ~5 s de latencia en las respuestas** |
| Idempotencia | Tabla `processed_events` (PK `event_id`); `registerIfAbsent` devuelve 1/0. La clave es el `eventId` del comando |
| Prerrequisito | Que existan `accounting.events`, `market.events` y sus `.DLT` en el broker compartido (**NO VERIFICADO**) |

## 5. Lo que Accounting espera recibir de Mercado

| Topic | `eventType` | Estado |
|---|---|---|
| `accounting.events` | `HOLD_CREATE_REQUESTED`, `HOLD_INCREASE_REQUESTED`, `HOLD_CONFIRM_REQUESTED`, `HOLD_RELEASE_REQUESTED` | Código y tests listos; **apagado** (`app.holds.commands.enabled=false`) y no encendible: no hay implementación de producción de `HoldReplyResender` y `CoinHoldsConfiguration` la exige al activar el flag. Con el flag apagado el dispatcher **ignora el mensaje y confirma el offset**: Mercado no recibe respuesta |
| `market.events` | `ITEM_CONFIRMED` | **Activo** (`app.inventory.writer.enabled=true`), pero con catálogo placeholder |
| `market.events` | `LIFE_PURCHASE_CONFIRMED` | **No existe en `develop`**; solo en la rama `feature/lives-purchase-credit` (flag `app.lives.purchase.enabled`) |

### Payloads y reglas

- **`HOLD_CREATE_REQUESTED`**: `studentId` (≤ 36), `courseId` (≤ 36, id del curso-cohorte), `orderId` (**UUID canónico**; otro formato ⇒ `HOLD_REJECTED / MALFORMED_COMMAND`), `orderType` (`DIRECT_PURCHASE` o `AUCTION_BID`), `amount` (decimal > 0, hasta 2 decimales, precisión 12), `ttlSeconds` (**obligatorio y > 0 para `AUCTION_BID`**; se ignora en `DIRECT_PURCHASE`).
- **`HOLD_INCREASE_REQUESTED`**: `holdId` (UUID), `newTotalAmount` (el **nuevo total**, no una diferencia; debe ser estrictamente mayor). Solo `AUCTION_BID`.
- **`HOLD_CONFIRM_REQUESTED`**: solo `holdId`; el monto y el tipo salen del hold guardado; los campos extra se ignoran.
- **`HOLD_RELEASE_REQUESTED`**: `holdId` y `releaseReason` ∈ {`AUCTION_LOST`, `AUCTION_CANCELLED`, `PURCHASE_NOT_COMPLETED`}. `TTL_EXPIRED` y `ACCOUNT_DEACTIVATED` desde Mercado ⇒ `MALFORMED_COMMAND`. **No existe liberar por `orderId`** (solo por `holdId`).
- **Envelope de los comandos de hold:** `eventType` desconocido, `eventId` que no sea UUID, o `producer` vacío o > 60 caracteres ⇒ el mensaje va a DLT. Los DTO son tolerantes a campos desconocidos.
- **`ITEM_CONFIRMED`**: `studentId`, `courseId`, `orderId` (≤ 36, no exige UUID), `catalogItemId` (≤ 36), `itemName` (≤ 50), `itemType` (≤ 50), `effect` (≤ 50). **No lleva** cantidad, alcance, cargas, multiplicador, duración ni vencimiento. Una confirmación crea **una** instancia; N unidades exigen N eventos con `eventId` distintos.

## 6. Lo que Accounting publica (todo en `accounting.events`)

### Respuestas de holds
Cada respuesta lleva `correlationId` = `eventId` del comando y un `eventId` nuevo propio (guardado en `processed_events.result_payload`).

| Evento | Payload |
|---|---|
| `HOLD_CREATED` | `correlationId, holdId, studentId, courseId, orderId, orderType, amount, status (PENDING), expiresAt, createdAt, updatedAt` |
| `HOLD_INCREASED` | `correlationId, holdId, studentId, courseId, orderId, orderType, amount (nuevo total), status, expiresAt (sin cambios), updatedAt`. **Conjunto de campos propuesto por Accounting, sin confirmar con Mercado** |
| `HOLD_CONFIRMED` | `correlationId, holdId, studentId, courseId, orderId, amount, orderType` (sin `ledgerEntryId`) |
| `HOLD_RELEASED` | `correlationId, holdId, studentId, courseId, orderId, orderType, amount, status (RELEASED), releaseReason, releasedAt` |
| `HOLD_REJECTED` | `correlationId, holdId (o null), reason, message`; `holdId` es `null` si el `create` fue rechazado o el `holdId` vino mal formado |

**`reason`** (`HoldRejectionReason`): `ACCOUNT_NOT_FOUND`, `ACCOUNT_INACTIVE`*, `HOLD_ALREADY_EXISTS`, `HOLD_NOT_FOUND`, `INVALID_HOLD_STATE`, `INSUFFICIENT_BALANCE`, `INVALID_AMOUNT`, `INVALID_ORDER_TYPE`*, `INVALID_TTL`*, `MALFORMED_COMMAND`*. (\* pendientes de confirmar con Mercado según `persona-4-entrega-c.md`.) Un débito que dejaría saldo negativo también se informa como `INSUFFICIENT_BALANCE`.

### Hechos contables y de inventario
- **`BALANCE_DEBITED`** (al confirmar un hold): `accountId, studentId, courseId, ledgerEntryId, amount (negativo), totalBalance, movementType (DIRECT_PURCHASE_DEBIT | AUCTION_WIN_DEBIT), sourceReferenceId (= orderId), description, createdDatetime`.
- **`ITEM_CREDITED`** (al acreditar un ítem): `studentId, courseId, itemInstanceId, itemName, itemType, sourceReferenceId (= orderId)`. Su `eventId` se deriva del de `ITEM_CONFIRMED`. **No tiene `correlationId`**: Mercado solo puede correlar por `sourceReferenceId`.
- Otros (no son de Mercado): `ITEM_CONSUMED` (desafíos), `ITEM_EQUIP_RESULT` / `ITEM_UNEQUIP_RESULT` (Roadmap).

### Lo que **no** publica
`HOLD_EXPIRED` (no hay job de expiración), cualquier rechazo o error de `ITEM_CONFIRMED`, `LIFE_CREDITED` por compra (no está en `develop`) y la respuesta a las reversiones de ADMIN (`COIN_LEDGER_REVERSAL_REQUESTED` está propuesta, no escrita en el outbox).

## 7. REST

Base `/api/accounting`. La identidad viene del Gateway: usuario (`X-Principal-Type: user`, roles `STUDENT`, `PROFESSOR`, `ADMIN`) o **servicio** (`X-Principal-Type: service` con `X-Service-Scopes` que contenga el literal `MS`, que se convierte en el rol `MS`; sin `MS` el llamador queda sin autenticar). Los errores usan el cuerpo propio `ErrorApi` (`timestamp, status, error, message, path, validationErrors`), **no** RFC 9457.

| Ruta | Quién puede | Nota para Mercado |
|---|---|---|
| `GET /courses/{courseId}/accounts/me` | `STUDENT` | Saldo total, reservado, disponible y vidas del propio alumno; solo con `app.accounts.queries-enabled` (en dev, `true`) |
| `GET /courses/{courseId}/accounts/{studentId}` | `ADMIN`, `PROFESSOR` | **`MS` no puede leer un saldo** |
| `GET /courses/{courseId}/accounts` | `MS` | Solo resumen de ranking (vidas perdidas), sin saldos |
| `GET .../accounts/{studentId}/equip-summary` | `MS`, `ADMIN`, `STUDENT` (propio) | **Única lectura de inventario permitida a servicios**: ítems con id, nombre, tipo y estado; no trae `catalogItemId`, `orderId` ni `effect` |
| `GET .../accounts/me/items`, `.../{studentId}/items`, `.../items-history` | `STUDENT` / `ADMIN` | Solo con `app.inventory.queries.enabled` (activo en dev) |
| `GET .../coins-movements`, `GET .../coins-reconciliation` | `STUDENT`/`ADMIN` | Historial y reporte; no son para Mercado |
| `POST /courses/{courseId}/challenge-reservations`, `GET /life-holds/{lifeHoldId}` | `MS` (Motor) | Solo desafíos y life holds |

**Que no existe** (verificado listando todos los `@*Mapping`): crear, confirmar, liberar, aumentar o **consultar un hold de monedas**; `GET /holds/{holdId}` (propuesto en el modelo como "contingencia para Mercado y Motor"; solo se construyó el equivalente de life holds); `GET .../students/me/coin-holds`; cualquier endpoint de **acreditar, otorgar o consumir ítems**; leer el saldo con rol `MS`; comprar vidas.

## 8. Dominio (lo que condiciona a Mercado)

### Hold de monedas (`coin_holds`)
- Estados: `PENDING → COMMITTED | RELEASED`; ambos finales. Solo `PENDING` acepta transiciones.
- **`UNIQUE(account_id, order_id)` sin importar el estado:** una orden es dueña de **un** hold para siempre; tras un `release` ese `orderId` no se puede reutilizar en esa cuenta. Un reintento con otro `eventId` no cambia esto (`create` ⇒ `HOLD_ALREADY_EXISTS`).
- **Crear:** cuenta existente y activa, sin hold previo para `(cuenta, orderId)`, `amount > 0` y `total − reservado ≥ amount`; suma a `reservado`. TTL: `DIRECT_PURCHASE` = ahora + `app.holds.direct-purchase-ttl-seconds` (300 s); `AUCTION_BID` = ahora + `ttlSeconds` (sin máximo).
- **Aumentar:** solo `AUCTION_BID` y `PENDING`, nuevo total estrictamente mayor, `total − reservado ≥ diferencia`; **no extiende el vencimiento**.
- **Confirmar:** en una sola actualización `total −= monto` y `reservado −= monto`, y asienta el ledger (`DIRECT_PURCHASE_DEBIT` o `AUCTION_WIN_DEBIT`, `sourceReference = orderId`). Es lo único que escribe el ledger. **No verifica `expiresAt`**: un hold vencido igual se puede confirmar o aumentar.
- **Liberar:** descuenta lo reservado, sin asiento.
- El bloqueo de fila de la cuenta serializa todos los holds de un alumno: la suma de holds `PENDING` nunca supera el saldo disponible; se admiten varios holds concurrentes por cuenta (uno por `orderId`).
- Sin `@Version` optimista (planificado).

### Cuenta y ledger
`student_accounts` (única por `(studentId, courseCohortId)`): `total_balance` (puede quedar negativo por reversiones de ADMIN), `reserved_balance ≥ 0`, vidas. `disponible = total − reservado` (derivado). `coins_ledgers` es de solo agregar; tipos de movimiento: `CHALLENGE_CREDIT`, `DIRECT_PURCHASE_DEBIT`, `AUCTION_WIN_DEBIT`, `COINS_REVERSAL`. **No hay reembolso de una compra ya confirmada** iniciado por Mercado: solo una reversión de ADMIN por Kafka, que se rechaza con `REVERSAL_BLOCKED_BY_PENDING_HOLDS` si dejaría el total por debajo de lo reservado.

### Inventario (`inventory_items`)
- **Una fila por unidad** (no un contador). Campos: `catalog_item_id`, `item_name`, `item_type` y `item_effect` (**texto libre sin validar**; el comentario de la migración lista `SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE`, `STREAK_FREEZE`, `LOOT_CHEST`), `applicable_challenge_scope` (**siempre `null`**), `order_id` (sin restricción de unicidad), `status` (`AVAILABLE`, `EQUIPPED`, `RESERVED`, `CONSUMED`), `max_charges`, `remaining_uses`.
- **No tiene** fecha de vencimiento, vigencia, duración, cantidad, multiplicador, modo ni regla de consumo.
- Crear un ítem exige que la cuenta exista y esté activa, y que `catalogItemId` ∈ {`ITEM-PLACEHOLDER-1`, `-2`, `-3`} (1, 2 y 3 cargas); cualquier otro ⇒ 3 reintentos y `market.events.DLT`, **sin avisar**. Un `effect`, `itemName` o `itemType` vacío también falla (aunque el AsyncAPI marca `effect` como opcional).
- **Solo `SHIELD` tiene lógica** (liquidación de desafíos). No hay lógica de `BOOST_*` ni de `LIFE`.
- Estados: `AVAILABLE ⇄ EQUIPPED` (comandos de Roadmap por Kafka) → `RESERVED` (Motor, al iniciar un desafío) → `AVAILABLE` o `CONSUMED` (al llegar `remaining_uses` a 0).

### Vidas
`student_accounts.current_lives / reserved_lives`; parámetro `maxLives` (PAR-12, contrato con Backoffice). El crédito por compra existe como `LIFE_PURCHASE_CREDIT` en el ledger de vidas, pero su entrada desde Mercado (`LIFE_PURCHASE_CONFIRMED`) está **solo en una rama sin PR**. En esa rama el tope se **trunca en silencio** (`aplicadas = min(cantidad, max(0, maxLives − currentLives))`) y Mercado se enteraría solo por `LIFE_CREDITED.livesAwarded`. En `develop` no hay ningún vínculo con "US-142".

## 9. Diferencias entre la documentación de Accounting y su código

| Tema | Lo que dice el doc | Lo que hace el código |
|---|---|---|
| Topic de Mercado | `market.orders.events` (modelo) | `market.events` |
| Topics de holds | `accounting.holds.commands/events` (`persona-4`, `persona-8`) | `accounting.events` para comandos y respuestas |
| Productor de Mercado | `tema-09-mercado`, `topic-09-marketplace` | No lo valida (solo no vacío; ≤ 60 caracteres en holds) |
| `receiveHoldCommands` | "implemented" (AsyncAPI) | Apagado y sin resender |
| `orderId` | ejemplos como `ord-1123` | UUID canónico obligatorio |
| Tipo/efecto de ítem | enums `SHIELD\|BOOST_XP\|BOOST_COINS\|LIFE` y `ABSORB_FAILURE\|XP_MULTIPLIER\|COIN_MULTIPLIER` | texto libre `VARCHAR(50)` sin validar |
| Liberar por orden | `releaseHoldsByOrder(orderId)` documentado | no construido; solo `findPendingIdsByOrder` sin uso |
| Expiración | `HoldExpirationScheduler` + `HOLD_EXPIRED` documentados | no construidos; solo `findExpiredPendingIds` sin uso |
| `GET /holds/{holdId}` | propuesto como contingencia | no existe |

## 10. Pendientes de Accounting que afectan a Mercado

1. **Habilitar los comandos de hold:** implementar `HoldReplyResender` de producción (dueño en sus docs: persona 8, entrega D2) y activar el flag. Bloquea todo.
2. **Catálogo real de ítems:** reemplazar `InventoryCatalog` (comentario en el código: "completar cuando se compartan los números reales").
3. **Errores de `ITEM_CONFIRMED`:** hoy van a DLT sin evento de vuelta y sin `correlationId` en `ITEM_CREDITED`.
4. **Expiración de holds** y evento `HOLD_EXPIRED`; validar `expiresAt` al confirmar.
5. **Liberación por orden / en lote** (necesaria para subastas).
6. **Desactivación de cuenta:** `AccountCoinReservationsPort` **no tiene implementación**, así que la baja de un alumno falla (va a DLT) y nunca libera holds (`HOLD_RELEASED / ACCOUNT_DEACTIVATED` no ocurre).
7. **Compra de vidas:** mergear `feature/lives-purchase-credit` o redefinir el tope.
8. Razones y campos **sin confirmar con Mercado:** `ACCOUNT_INACTIVE`, `INVALID_ORDER_TYPE`, `INVALID_TTL`, `MALFORMED_COMMAND`, payload de `HOLD_INCREASED` y máximo de `ttlSeconds`.
9. Concurrencia optimista (`@Version`) del hold.

## 11. Qué implica para Mercado

- El contrato de mensajes **de Accounting** es el de plataforma (`accounting.events` / `market.events`, envelope de 6 campos, `orderId` UUID). Mercado debe adaptarse a él; los topics propios de Mercado no existen en el broker.
- Mercado **no puede leer saldo ni holds** de Accounting por REST hoy; solo por los eventos de respuesta.
- La dirección de **liquidación en un solo comando** (compra directa) no existe todavía en Accounting; se apoya en bloques ya implementados (`LedgerWriteService.appendDebitEntry`, `InventoryItemService.createItemInstance`, registro de comandos procesados y outbox). Ver [`contrato-integracion-mercado-accounting.md`](./contrato-integracion-mercado-accounting.md) y el [taller de decisiones](../../mercado/estado-actual/taller-decisiones.html).
- Para subastas hay que cerrar primero los puntos 1, 5 y 6 de §10.

## 12. Verificado y no verificado

**Verificado en código (`develop` @ `3013f6c`):** flags y su valor por defecto, ausencia de `HoldReplyResender` de producción, catálogo placeholder, dispatcher por topic y `eventType`, `orderId` UUID, unicidad del hold por `(cuenta, orderId)`, servicios reutilizables, ausencia de implementación de `AccountCoinReservationsPort`, ramas sin mergear.
**No verificado:** que los topics `accounting.events`, `market.events` y sus DLT existan en el broker compartido; el rendimiento real del listener único con el relay de 5 s; el comportamiento de las ramas sin mergear más allá de leer sus archivos; que la historia US-142 corresponda a la regla de tope de vidas (no aparece en ese repo); la ejecución real de la aplicación.
