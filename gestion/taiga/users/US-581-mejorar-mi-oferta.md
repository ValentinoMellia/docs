# [G11 — Mejorar mi oferta]

> **Taiga Ref:** #581 | **ID:** 9549021
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Mejorar mi oferta

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO que ya ofertó
*   **Quiero**: subir el monto de mi oferta
*   **Para**: volver a ir ganando la subasta

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: la oferta solo puede subir. El Banco amplía la retención; solo tiene que verificar que el alumno tenga la diferencia. No se puede bajar ni retirar una oferta (RF-INT-05).
*   [ ] Validaciones: el nuevo monto supera la oferta actual del alumno y la oferta más alta de la subasta.
*   [ ] Datos obligatorios: subasta, nuevo monto.
*   [ ] Performance (tiempos, volumen, límites): resultado en menos de 5 segundos.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo el dueño de la oferta puede mejorarla.
*   [ ] Accesibilidad (WCAG/teclado/lectores): se muestra la oferta anterior y la nueva antes de confirmar.
*   [ ] Otros: si la mejora falla, la oferta anterior sigue igual.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Si el Banco amplía la retención, la oferta se actualiza y se recalcula quién va ganando.
*   [ ] CA2: Si no alcanza para la diferencia, la oferta anterior sigue vigente con su retención.
*   [ ] CA3: No existe forma de bajar ni retirar una oferta; si se intenta, el sistema lo rechaza.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Mejorar una oferta existente

**Escenario 1: mejora válida**

*   **Dado**: que mi oferta es 850 y la más alta es 900
*   **Cuando**: subo mi oferta a 950
*   **Entonces**: el Banco me retiene 950 en total y vuelvo a ir ganando

**Escenario 2: no alcanza para la diferencia**

*   **Dado**: que mi oferta es 850 y me quedan 50 monedas libres
*   **Cuando**: intento subirla a 950
*   **Entonces**: se rechaza y mi oferta de 850 sigue vigente

**Escenario 3: intento de retirar la oferta**

*   **Dado**: que tengo una oferta en una subasta abierta
*   **Cuando**: intento bajarla o retirarla
*   **Entonces**: el sistema muestra "Las ofertas solo pueden aumentar"

* * *

## Prototipo

*   **Mock API / Swagger**: `PUT /api/v1/market/auctions/{id}/bids` — eventos `HOLD_INCREASE_REQUESTED` / `HOLD_INCREASED`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: G08 (operación para ampliar una retención).
*   Impacto en datos / migraciones: se actualiza `auction_bid` y se guarda el historial de montos.
*   Riesgos y mitigación (opcional): dos mejoras casi simultáneas → se procesan en orden, una por vez por subasta.


