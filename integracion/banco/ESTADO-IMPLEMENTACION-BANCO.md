# Estado real de implementación — Integración Mercado ↔ Banco

> **Estado: vigente.** Este documento describe lo que **hoy existe implementado en el código** del repo `tpi-market` (commit `455fc3f`, 27/09/2026), no un contrato ideal o propuesto. Para el diseño objetivo de la integración ver [`flujo-mercado-inventario.md`](flujo-mercado-inventario.md) (saga consolidada) y [`contrato-integracion-mercado-accounting.md`](contrato-integracion-mercado-accounting.md) (contrato Kafka async propuesto) — ambos están, a la fecha de este documento, **desalineados con lo implementado** (ver Sección 4).

## 1. Qué está implementado y confirmado hoy

La integración con Banco es **sincrónica**, no por Kafka, y hoy corre contra un **cliente mock** (`MockBankHoldClient`), no contra el microservicio real de Banco. Lo mismo aplica al cliente de Inventario (`MockInventoryItemProvisionClient`).

Flujo real (tareas US-138 T03 y T04, ambas completas y mergeadas):

1. **T03 — Pedido de hold.** Al crear la orden, Mercado transiciona a `HOLD_REQUESTED` y le pide a Banco (`BankHoldClient.requestHold`) que retenga las monedas (`currency = "GOLD_COIN"`, `ttlSeconds = 300`, `appliedPrice` de la orden). La llamada corre **fuera de cualquier transacción de base de datos activa** (patrón TX-A → llamada externa → TX-B), para no dejar una transacción abierta esperando una respuesta de red.
   - Hold otorgado → `HOLD_REQUESTED → HOLD_GRANTED`, se persiste el `holdId`.
   - Hold rechazado (`INSUFFICIENT_FUNDS` o cualquier código no reconocido, por seguridad) → `HOLD_REQUESTED → REJECTED_INSUFFICIENT_FUNDS`.
   - Banco no responde o falla → `HOLD_REQUESTED → EXPIRED`, se libera el stock si la oferta tenía stock reservado.
   - Resultados tardíos o duplicados sobre una orden que ya no está en `HOLD_REQUESTED` se ignoran (idempotencia por guardia de estado, no se lanza excepción).

2. **T04 — Provisión del ítem.** Si el hold fue otorgado, se dispara automáticamente `OrderItemProvisionService.requestProvision`, que pide a Inventario que acredite el ítem:
   - Acreditado → `ITEM_PROVISION_REQUESTED → ITEM_PROVISIONED` (estado *no terminal*: solo falta confirmar el cobro).
   - Falla o Inventario no responde → se libera el hold de Banco (`releaseHold`, best-effort) y se libera el stock reservado, orden → `CANCELLED`.

Máquina de estados real (9 estados, ya implementada en código y tests — **el archivo `openspec/specs/order-state-machine/spec.md` de `tpi-market` todavía no fue actualizado a esta versión**, sigue mostrando la tabla vieja de 7 estados de T03):

```
CREATED → HOLD_REQUESTED
HOLD_REQUESTED → HOLD_GRANTED | REJECTED_INSUFFICIENT_FUNDS | EXPIRED
HOLD_GRANTED → ITEM_PROVISION_REQUESTED | EXPIRED
ITEM_PROVISION_REQUESTED → ITEM_PROVISIONED | CANCELLED
ITEM_PROVISIONED → CONFIRMED   ← todavía no implementado (T05)
```

## 2. Decisiones de diseño ya tomadas (no rediscutir sin motivo)

- El hold de Banco se libera **antes** de marcar la orden como `CANCELLED`, para que el mensaje al alumno ("se te devolvieron las monedas") sea siempre veraz.
- Los `reasonCode` que Mercado le manda a Banco al liberar un hold son códigos **propios de Mercado** (`ITEM_PROVISION_FAILED`, `ITEM_PROVISION_UNAVAILABLE`, `OFFER_NOT_FOUND`) — no se reenvía el código crudo de Inventario, porque el vocabulario que acepta Banco todavía no está confirmado (ver Sección 5).
- **Banco e Inventario son dos microservicios separados** (confirmado con Grupo 08 el 26/09/2026) — esto corrige la decisión #13 de `CONTEXTO-MERCADO-SPRINT1.md` §12, que los daba por fusionados en "Grupo 12 (Banco)". Ver Sección 3 para la corrección formal pendiente.
- El payload de provisión del ítem (`itemPayload`) es híbrido: campos tipados + un mapa de atributos libres, precisamente porque el schema definitivo con Grupo 12 todavía no está cerrado.
- Ya se dejaron "seams" preparados (`applyHoldResult`, `applyProvisionResult`) para que un futuro listener de Kafka los llame directo, sin pasar por el flujo síncrono actual — pero es solo una preparación estructural, no hay ningún consumidor Kafka implementado.

## 3. Qué falta implementar con Banco

1. **Integración real con Banco**, reemplazando `MockBankHoldClient`: transporte (REST/Kafka — todavía no decidido, ver Sección 4), autenticación de servicio, timeout, `Idempotency-Key`, y reconciliación para el caso "Banco respondió tarde pero sí llegó a provisionar".
2. **T05 — Confirmación del cobro** (`ITEM_PROVISIONED → CONFIRMED`): no tiene ni una carpeta de change en OpenSpec, ni un commit, ni una historia de Taiga iniciada. Es el paso que falta para cerrar la saga — hoy una orden puede quedar en `ITEM_PROVISIONED` sin que nada la confirme.
3. **Vocabulario de `reasonCode` para `releaseHold`**: qué códigos acepta Banco realmente al liberar un hold — hoy es una pregunta abierta sin confirmar con Grupo 08.
4. **Persistencia de `inventoryItemId`**: hoy solo se logea, no se guarda en la orden. Si T05 (confirmación) o un futuro flujo de reembolso lo necesitan, falta agregar la columna.

## 4. Qué falta definir (decisiones de arquitectura, no de código)

1. ~~**Nombre y cantidad de servicios**~~ — **Resuelto (27/09/2026).** Historia real: Inventario (la mochila) se transfirió de Mercado a Banco; después el equipo que quedó a cargo de Banco dividió ese dominio combinado en **dos microservicios**: **Accounting** (Tema 08) — ledger financiero, Balance Hold, exclusivamente — y un microservicio de **Inventario** separado, que es quien custodia `student_inventory`. Confirmado con Grupo 08.
   - **Consecuencia:** `contrato-integracion-mercado-accounting.md` describe a Accounting como responsable de instanciar el ítem en la mochila (§1 paso 3) — eso ya no es así tras el split. Se agregó una nota de advertencia en ese documento (27/09/2026) señalando el punto exacto a corregir con su autor.
2. **Cuál de los dos contratos escritos es el que se va a implementar en T07** (el paso Kafka async, todavía sin iniciar en ningún repo), ahora que se sabe que son 3 partes (Mercado, Accounting, Inventario) y no 2:
   - `flujo-mercado-inventario.md` ya modela Banco e Inventario como integraciones separadas — más alineado con el split real.
   - `contrato-integracion-mercado-accounting.md` describe solo el tramo Mercado↔Accounting (eventos `PURCHASE_SETTLEMENT_*`) y, una vez corregido el punto de la nota, necesita un tramo equivalente Mercado↔Inventario (o Accounting↔Inventario) que hoy no está escrito en ningún lugar.
   - El código real de `tpi-market` no conoce ninguno de los dos contratos concretos todavía — sigue pendiente decidir/escribir el contrato final de 3 partes antes de que alguien implemente T07.
3. **Corrección formal de `CONTEXTO-MERCADO-SPRINT1.md` §12** (decisión #13): agregar una entrada §12-F que registre el split confirmado (punto 1), corrigiendo la fusión "Grupo 12 (Banco)" que quedó mal ahí.
4. **Snapshot vs. lectura en el momento de las mecánicas del ítem**: si T07 introduce async real, ¿qué pasa si un profesor edita la oferta entre que se pidió la provisión y que se procesa? Hoy no hay una decisión tomada sobre si congelar un snapshot al momento de compra.

## 5. Documentos relacionados

| Documento | Rol respecto a este |
|---|---|
| [`flujo-mercado-inventario.md`](flujo-mercado-inventario.md) | Especificación consolidada del diseño objetivo de la saga (no necesariamente lo implementado hoy) |
| [`contrato-integracion-mercado-accounting.md`](contrato-integracion-mercado-accounting.md) | Contrato Kafka async propuesto para T07 — sin confirmar contra el código, ver Sección 4 |
| `CONTEXTO-MERCADO-SPRINT1.md` §12 | Tiene una decisión (#13) que este documento señala como corregida por Grupo 08 el 26/09 (pendiente de formalizar como §12-F) |
