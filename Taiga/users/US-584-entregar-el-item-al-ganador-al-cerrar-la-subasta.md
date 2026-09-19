# [G11 — Entregar el ítem al ganador al cerrar la subasta]

> **Taiga Ref:** #584 | **ID:** 9549024
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Entregar el ítem al ganador al cerrar la subasta

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO que ganó una subasta
*   **Quiero**: que al terminar se me cobre lo que ofrecí y reciba el ítem
*   **Para**: poder usarlo en el curso

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: Mercado cierra la subasta al llegar la hora de fin y elige la oferta más alta; si hay empate, gana la que se confirmó primero. Antes de cobrarle nada al ganador, Mercado le pide a Grupo 12 que acredite el ítem (`ITEM_PROVISION_REQUESTED`); solo si Grupo 12 confirma la acreditación (`ITEM_PROVISIONED`) se le pide al Banco que confirme el débito (`HOLD_CONFIRM_REQUESTED`). El orden nunca es al revés: no se cobra sin haber corroborado antes que el ítem se puede entregar. Se guarda el precio final.
*   [ ] Validaciones: la subasta se cierra una sola vez, aunque haya más de una instancia de Mercado corriendo.
*   [ ] Datos obligatorios: ganador, monto final, fecha de cierre.
*   [ ] Performance (tiempos, volumen, límites): el cierre empieza como máximo 1 minuto después de la hora de fin.
*   [ ] Seguridad (roles, permisos, datos sensibles): el cierre lo hace el sistema; nadie puede forzarlo a mano.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el aviso al ganador se puede leer con lector de pantalla.
*   [ ] Otros: se avisa al ganador (evento `AUCTION_AWARDED`) y se informa a Backoffice.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: El débito se confirma (`HOLD_CONFIRM_REQUESTED`) solo después de que Grupo 12 confirmó que el ítem se puede acreditar (`ITEM_PROVISIONED`) — nunca antes.
*   [ ] CA2: Con dos instancias de Mercado, la subasta se cierra y se entrega una sola vez.
*   [ ] CA3: Si Grupo 12 no puede acreditar el ítem (`ITEM_PROVISION_FAILED`), el débito nunca se confirma — no hay nada que devolver porque nunca se cobró. Mercado reintenta la acreditación con backoff; si se agotan los reintentos, la subasta queda en `FAILED_SETTLEMENT` para revisión manual.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Cierre de subasta con ganador

**Escenario 1: entrega normal**

*   **Dado**: que la subasta terminó y mi oferta de 950 es la más alta
*   **Cuando**: Mercado cierra la subasta
*   **Entonces**: Grupo 12 acredita el ítem primero, el Banco me cobra 950 recién después, y recibo un aviso `AUCTION_AWARDED` de que gané

**Escenario 2: cierre simultáneo**

*   **Dado**: que dos instancias de Mercado intentan cerrar la misma subasta
*   **Cuando**: ambas lo intentan a la vez
*   **Entonces**: solo una la cierra y la otra no hace nada

**Escenario 3: falla la acreditación del ítem**

*   **Dado**: que soy el ganador y Grupo 12 no puede acreditar el ítem (`ITEM_PROVISION_FAILED`)
*   **Cuando**: Mercado agota los reintentos de acreditación
*   **Entonces**: nunca se me cobra (el hold nunca llega a `HOLD_CONFIRM_REQUESTED`) y la subasta queda en `FAILED_SETTLEMENT` para revisión manual

* * *

## Prototipo

*   **Mock API / Swagger**: eventos `ITEM_PROVISION_REQUESTED`, `ITEM_PROVISIONED` / `ITEM_PROVISION_FAILED`, `HOLD_CONFIRM_REQUESTED`, `HOLD_CONFIRMED`, `AUCTION_AWARDED`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 5
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08, Notificaciones, Grupo 12 (dueño del inventario del ganador).
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: contrato `ITEM_PROVISION_REQUESTED`/`ITEM_PROVISIONED`/`ITEM_PROVISION_FAILED` a formalizar bilateralmente con Grupo 12 (bloquea esta historia); G08 (confirmación del débito, siempre después de la acreditación).
*   Impacto en datos / migraciones: `auction` guarda ganador, monto final y fecha de cierre; campo `version` para evitar cierres dobles; estado `FAILED_SETTLEMENT` para reintentos manuales/cron.
*   Riesgos y mitigación (opcional): que se entregue dos veces → control de versión en la subasta y un solo proceso de cierre activo. Que se confirme el débito antes de corroborar la acreditación (el bug que tenía esta historia) → el orden `CREDITING_ITEM` antes de `CONFIRMING_LEDGER` es la máquina de estados canónica (CONTEXTO §6) y no debe invertirse.


