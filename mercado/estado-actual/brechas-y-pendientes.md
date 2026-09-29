# Mercado: brechas y pendientes consolidados

> **Estado al 29/09/2026** · consolida lo verificado en [`compra.md`](./compra.md), [`gestion-de-tienda.md`](./gestion-de-tienda.md), [`subasta.md`](./subasta.md) y [`estado-integracion-mercado-accounting.md`](../../integracion/banco/estado-integracion-mercado-accounting.md). Prioridades propuestas (no acordadas): **P0** bloquea el flujo real, **P1** integridad/robustez, **P2** deuda.

## P0: sin esto no hay compra real (ni subasta)

| # | Pendiente | Dueño | Dependencia |
|---|---|---|---|
| 1 | Habilitar los comandos de hold en Accounting: implementar `HoldReplyResender` de producción y activar `app.holds.commands.enabled` | Accounting | Bloquea todo lo demás |
| 2 | Mercado usa los topics de plataforma: hold → `accounting.events`, `ITEM_CONFIRMED` y eventos de orden → `market.events` (hoy `accounting.holds.*`, `inventory.items.*`, `market.orders.events`, que no existen en el broker) | Mercado | Verificar que `accounting.events` y `market.events` (+ `.DLT`) existan |
| 3 | `orderRef` UUID por orden, usado como `orderId` (hoy `"42"` ⇒ `MALFORMED_COMMAND`) | Mercado | — |
| 4 | Reemplazar `ITEM_PROVISION_REQUESTED/ITEM_PROVISIONED` por `ITEM_CONFIRMED` → `ITEM_CREDITED` (cierra Q5) | Mercado + Accounting | Decisión D2 |
| 5 | Catálogo real en Accounting (dejar `ITEM-PLACEHOLDER-*`), `charges` en `ITEM_CONFIRMED`, `correlationId` en `ITEM_CREDITED` y evento de error | Accounting | #4 |
| 6 | Que el contexto arranque bajo `transport=kafka` (bean `BankHoldQueryClient`) y que lea `SPRING_KAFKA_BOOTSTRAP_SERVERS` | Mercado | — |
| 7 | Correr `KafkaSagaIntegrationTest` en verde (hoy nunca corrió) y agregar prueba de contrato contra los payloads reales de Accounting | Mercado | #2, #3 |
| 8 | Cliente real de matrícula/docente (hoy mocks en todos los perfiles) | Mercado + Cursos | Contrato de Cursos |

## P1: integridad y robustez

| # | Pendiente | Dueño |
|---|---|---|
| 9 | **Bug `units_sold` nunca se incrementa**: editar stock tras vender devuelve unidades al stock (sobreventa) | Mercado |
| 10 | Vencer órdenes en `HOLD_REQUESTED`, `ITEM_PROVISION_REQUESTED`, `ITEM_PROVISIONED` y `CREATED` tras un fallo de `requestHold`; con el reconciliador apagado hoy quedan trabadas | Mercado |
| 11 | Dueño de la expiración de holds (job + `HOLD_EXPIRED` en Accounting); hoy nadie expira ni valida `expiresAt` al confirmar | Accounting (decisión D4) |
| 12 | Consulta de estado de hold (`GET /api/accounting/holds/{holdId}`) o eliminar la dependencia del reconciliador | Accounting / Mercado (D5) |
| 13 | **Compra de vidas / tope (US-142):** definir si es ítem o vida de cuenta; hoy Accounting trunca en silencio y Mercado nunca produce `LIFE_CAP_REACHED` (issues #13 y #15 abiertos) | PO + ambos (D1) |
| 14 | Mapear cada razón de `HOLD_REJECTED` (hoy todo ⇒ `REJECTED_INSUFFICIENT_FUNDS`) y el `reasonCode` de aprovisionamiento (se descarta) | Mercado |
| 15 | Autorización en el servicio de gestión: sacar `contains()` sobre headers y los atajos con header vacío | Mercado |
| 16 | Outbox: el relay se frena ante la primera fila fallida; agregar tope de reintentos/poison, índice y purga de `outbox_events` y `processed_events` | Mercado |
| 17 | Mapear `BankHoldUnavailableException`, `InventoryItemProvisionUnavailableException` y `ObjectOptimisticLockingFailureException` (hoy `500`) | Mercado |
| 18 | Liberación de holds al desactivar cuenta (`AccountCoinReservationsPort` sin implementación) | Accounting |

## P2: deuda y calidad

- **Tienda:** validar `templateId`/`itemType` y multiplicador mínimo al publicar; chequear vencimiento en el detalle; decidir sobre baja lógica y cohortes; retirar los restos de `itemValidityDays` (entidad, DTO, validador) y archivar/actualizar `openspec/changes/item-validity-duration`.
- **CI:** los PRs a `develop` solo validan el nombre de rama; `mvn verify` corre contra `main`/`release`. Sumar tests a los PRs a `develop`. La rama por defecto del repo es `main` y está 222 commits atrás de `develop`.
- **Esquema:** hoy lo genera Hibernate (`ddl-auto`); sin Flyway/Liquibase no hay historial de cambios de base (Accounting sí usa Flyway).
- **Consistencia:** `producer` (`market-service` vs `tema-09-mercado`), `groupId` fijo del listener de holds, puerto de Mercado (`8084` en la app vs el registro de plataforma).
- **OpenSpec:** ~9 cambios terminados sin archivar; el directorio vacío `catalog-offer-publish-edit` ya está archivado; `config.yaml` menciona "No Kafka" y un unique constraint que ya se entregó.
- **Documentación generada:** regenerar javadoc (faltan 17 clases) y el `swagger.json` (placeholders `@project.name@`, contacto "Joe Doe", `servers: localhost:8080`); el diagrama de clases sigue mostrando un `PingController` que no existe. Detalle en la auditoría del PR de estructura.

## Subasta (Sprint 02): prerrequisitos

Además de todo el P0:

1. Ajustar los contratos de `mercado/subastas/03` a lo que Accounting acepta (`AUCTION_LOST`/`AUCTION_CANCELLED`, un comando por `holdId`; sin `AUCTION_REFUND` ni evento en lote).
2. Generalizar en Mercado el modelo de saga que hoy asume "agregado = orden" (`orderId` Long, `aggregateType="ORDER"`, `HoldEventDto` sin `orderType`).
3. Definir liberación en lote y expiración de holds con Accounting, y la baja de alumno (US-588).
4. Extender `BankHoldClient` con `HOLD_INCREASE_REQUESTED`.

Ver [`subasta.md`](./subasta.md) §4–§7.

## Preguntas abiertas con dueño externo

| Pregunta | A quién | Estado |
|---|---|---|
| Q3: nombres de topics/campos de Inventario no confirmados por escrito | Grupo 12 / Accounting | Abierta (con el diseño de este documento, se resuelve adoptando `ITEM_CONFIRMED`) |
| Q5: ¿`ITEM_CONFIRMED` reemplaza al par `inventory.items.*`? | Accounting + revisor | Abierta, bloqueante |
| Q6: códigos de `effect` para `BOOST_XP`, `BOOST_COINS`, `LIFE` | Accounting / Motor | Abierta |
| ¿Existen `accounting.events`, `market.events` y sus DLT en el broker compartido? | Plataforma | No verificado |
| Puerto de Mercado en el registro de plataforma (8100 vs 8084) | Plataforma | Abierta |
