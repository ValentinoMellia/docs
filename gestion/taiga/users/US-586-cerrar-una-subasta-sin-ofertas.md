# [G11 — Cerrar una subasta sin ofertas]

> **Taiga Ref:** #586 | **ID:** 9549026
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Cerrar una subasta sin ofertas

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: que una subasta sin ofertas termine como "desierta"
*   **Para**: saber que el ítem no se entregó y poder volver a subastarlo

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: si al terminar no hay ninguna oferta válida, la subasta queda como desierta, nadie recibe el ítem y no se le pide nada al Banco (RF-INT-05).
*   [ ] Validaciones: las ofertas rechazadas no cuentan.
*   [ ] Datos obligatorios: resultado "desierta", fecha de cierre.
*   [ ] Performance (tiempos, volumen, límites): igual que un cierre normal.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo el profesor del curso ve el detalle.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el resultado se muestra como texto, no solo con color.
*   [ ] Otros: el profesor puede crear una nueva subasta con el mismo ítem.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Una subasta sin ofertas válidas queda como desierta.
*   [ ] CA2: No se genera ningún movimiento en el Banco.
*   [ ] CA3: El profesor ve el resultado "Desierta" en su listado.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Subasta desierta

**Escenario 1: sin ofertas**

*   **Dado**: que una subasta llegó a su hora de fin sin ofertas
*   **Cuando**: Mercado la cierra
*   **Entonces**: queda como desierta y nadie recibe el ítem

**Escenario 2: solo ofertas rechazadas**

*   **Dado**: que todas las ofertas se rechazaron por falta de monedas
*   **Cuando**: la subasta termina
*   **Entonces**: queda como desierta

**Escenario 3: el profesor la vuelve a lanzar**

*   **Dado**: que una subasta quedó desierta
*   **Cuando**: creo una nueva subasta con el mismo ítem
*   **Entonces**: el sistema me lo permite

* * *

## Prototipo

*   **Mock API / Swagger**: `GET /api/v1/market/auctions/{id}` (campo `result: DESERTED`)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 1
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: campo `result` en `auction`.
*   Riesgos y mitigación (opcional): —


