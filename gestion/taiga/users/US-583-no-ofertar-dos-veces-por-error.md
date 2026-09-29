# [G11 — No ofertar dos veces por error]

> **Taiga Ref:** #583 | **ID:** 9549023
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — No ofertar dos veces por error

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO con conexión inestable
*   **Quiero**: que si mi oferta se envía dos veces, cuente como una sola
*   **Para**: no terminar con monedas retenidas de más

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: cada intento de oferta lleva un código único. Si llega dos veces el mismo código, Mercado responde lo mismo que la primera vez y no le pide nada nuevo al Banco. Las respuestas repetidas del Banco también se ignoran.
*   [ ] Validaciones: el código único es obligatorio en cada oferta.
*   [ ] Datos obligatorios: código único del intento.
*   [ ] Performance (tiempos, volumen, límites): los códigos se guardan al menos 24 horas.
*   [ ] Seguridad (roles, permisos, datos sensibles): un código solo vale para el alumno que lo generó.
*   [ ] Accesibilidad (WCAG/teclado/lectores): no aplica.
*   [ ] Otros: el botón "Ofertar" se deshabilita mientras se procesa.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Dos envíos con el mismo código generan una sola oferta y una sola retención.
*   [ ] CA2: Una respuesta repetida del Banco no cambia nada.
*   [ ] CA3: El segundo envío recibe la misma respuesta que el primero.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Protección contra ofertas duplicadas

**Escenario 1: doble clic**

*   **Dado**: que hago doble clic en "Ofertar 850"
*   **Cuando**: llegan dos pedidos con el mismo código
*   **Entonces**: se registra una sola oferta y el Banco retiene 850 una sola vez

**Escenario 2: reintento después de perder señal**

*   **Dado**: que envié una oferta y perdí la señal antes de ver la respuesta
*   **Cuando**: la reenvío al recuperar la señal
*   **Entonces**: veo el resultado de mi oferta original, sin una nueva retención

**Escenario 3: respuesta repetida del Banco**

*   **Dado**: que Mercado ya recibió la confirmación del Banco para mi oferta
*   **Cuando**: llega otra vez la misma confirmación
*   **Entonces**: se ignora y la oferta no cambia

* * *

## Prototipo

*   **Mock API / Swagger**: encabezado `X-Idempotency-Key` en `POST` y `PUT /api/v1/market/auctions/{id}/bids`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 2
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: tabla `processed_request` (códigos ya procesados).
*   Riesgos y mitigación (opcional): —


