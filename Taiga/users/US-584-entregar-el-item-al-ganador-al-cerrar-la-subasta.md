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

*   [ ] Reglas de negocio: Mercado cierra la subasta al llegar la hora de fin y elige la oferta más alta; si hay empate, gana la que se confirmó primero. Le pide al Banco que cobre y, cuando el Banco confirma, pone el ítem en el inventario del ganador. Se guarda el precio final. Si falla la entrega del ítem, se le devuelven las monedas al ganador.
*   [ ] Validaciones: la subasta se cierra una sola vez, aunque haya más de una instancia de Mercado corriendo.
*   [ ] Datos obligatorios: ganador, monto final, fecha de cierre.
*   [ ] Performance (tiempos, volumen, límites): el cierre empieza como máximo 1 minuto después de la hora de fin.
*   [ ] Seguridad (roles, permisos, datos sensibles): el cierre lo hace el sistema; nadie puede forzarlo a mano.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el aviso al ganador se puede leer con lector de pantalla.
*   [ ] Otros: se avisa al ganador y se informa a Backoffice.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: El ítem se entrega solo después de que el Banco confirma el cobro.
*   [ ] CA2: Con dos instancias de Mercado, la subasta se cierra y se entrega una sola vez.
*   [ ] CA3: Si el ítem no se puede entregar, se devuelven las monedas al ganador y la subasta queda marcada para revisión.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Cierre de subasta con ganador

**Escenario 1: entrega normal**

*   **Dado**: que la subasta terminó y mi oferta de 950 es la más alta
*   **Cuando**: Mercado cierra la subasta
*   **Entonces**: el Banco me cobra 950, recibo el ítem en mi inventario y un aviso de que gané

**Escenario 2: cierre simultáneo**

*   **Dado**: que dos instancias de Mercado intentan cerrar la misma subasta
*   **Cuando**: ambas lo intentan a la vez
*   **Entonces**: solo una la cierra y la otra no hace nada

**Escenario 3: falla la entrega del ítem**

*   **Dado**: que el Banco ya me cobró
*   **Cuando**: falla el guardado del ítem en mi inventario
*   **Entonces**: se me devuelven las 950 monedas y la subasta queda marcada para revisión

* * *

## Prototipo

*   **Mock API / Swagger**: eventos `HOLD_CONFIRM_REQUESTED`, `HOLD_CONFIRMED`, `SUBASTA_ADJUDICADA`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 5
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08, Notificaciones, Backoffice G12.
*   Módulos afectados: Subastas, Inventario.
*   Otros equipos / aprobaciones: G08 (cobro de una retención y devolución de un cobro).
*   Impacto en datos / migraciones: `auction` guarda ganador, monto final y fecha de cierre; campo `version` para evitar cierres dobles.
*   Riesgos y mitigación (opcional): que se entregue dos veces → control de versión en la subasta y un solo proceso de cierre activo.


