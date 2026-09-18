# [G11 — Hacer una oferta en una subasta]

> **Taiga Ref:** #580 | **ID:** 9549020
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Hacer una oferta en una subasta

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO
*   **Quiero**: ofertar monedas en una subasta abierta
*   **Para**: intentar quedarme con el ítem

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: la oferta tiene que superar la oferta mínima y la oferta más alta del momento. El Banco retiene las monedas ofertadas; recién cuando el Banco confirma, la oferta cuenta. Mientras tanto se muestra "procesando tu oferta".
*   [ ] Validaciones, en este orden: la subasta está abierta → el alumno está matriculado en el curso → el monto supera la oferta más alta → el alumno tiene monedas suficientes en ese curso.
*   [ ] Datos obligatorios: subasta, monto.
*   [ ] Performance (tiempos, volumen, límites): el alumno ve el resultado en menos de 5 segundos; si el Banco no responde en ese tiempo, la oferta se rechaza sin retener nada.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo ALUMNOS matriculados; el alumno solo puede ofertar con sus propias monedas.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el resultado de la oferta se anuncia al lector de pantalla.
*   [ ] Otros: funciona desde el celular.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Si el Banco retiene las monedas, la oferta queda registrada y, si es la más alta, el alumno pasa a ir ganando.
*   [ ] CA2: Si no le alcanzan las monedas, la oferta se rechaza y su saldo no cambia.
*   [ ] CA3: Si el Banco no responde en 5 segundos, la oferta se rechaza con el mensaje "No se pudo procesar tu oferta, no se descontó nada. Intentá de nuevo".
*   [ ] Extras (opcional): Una oferta menor o igual a la más alta se rechaza antes de consultar al Banco.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Ofertar en una subasta

**Escenario 1: primera oferta válida**

*   **Dado**: que la subasta está abierta con oferta mínima de 800 y tengo 1.000 monedas en el curso
*   **Cuando**: oferto 850
*   **Entonces**: el Banco me retiene 850 monedas y aparezco ganando la subasta

**Escenario 2: monedas insuficientes**

*   **Dado**: que tengo 500 monedas disponibles
*   **Cuando**: oferto 850
*   **Entonces**: la oferta se rechaza con "No tenés monedas suficientes" y mi saldo sigue igual

**Escenario 3: oferta menor a la más alta**

*   **Dado**: que la oferta más alta es 900
*   **Cuando**: oferto 880
*   **Entonces**: el sistema me avisa que tengo que ofertar más de 900

* * *

## Prototipo

*   **Mock API / Swagger**: `POST /api/v1/market/auctions/{id}/bids` — eventos `HOLD_CREATE_REQUESTED` / `HOLD_CREATED`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 5
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08, Cursos y Matrícula G02.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: G08 (retención que dure toda la subasta, con 30 minutos de margen).
*   Impacto en datos / migraciones: tabla `auction_bid`.
*   Riesgos y mitigación (opcional): que el Banco libere la retención antes del cierre → la retención se pide con 30 minutos de margen sobre el tiempo restante.


