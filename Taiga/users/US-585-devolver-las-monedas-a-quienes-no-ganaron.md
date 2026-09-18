# [G11 — Devolver las monedas a quienes no ganaron]

> **Taiga Ref:** #585 | **ID:** 9549025
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Devolver las monedas a quienes no ganaron

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO que no ganó una subasta
*   **Quiero**: recuperar mis monedas retenidas apenas termina
*   **Para**: poder usarlas en otra cosa sin haber perdido nada

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: después de entregar el ítem al ganador, Mercado le pide al Banco que devuelva las monedas de todos los demás. Cada devolución queda anotada como pendiente hasta que el Banco la confirma. La subasta se da por terminada cuando se pidieron todas.
*   [ ] Validaciones: se devuelve el 100% de lo retenido, sin costo.
*   [ ] Datos obligatorios: alumno, monto, estado de la devolución.
*   [ ] Performance (tiempos, volumen, límites): 50 devoluciones pedidas en menos de 1 minuto; de ser posible, en un solo pedido al Banco.
*   [ ] Seguridad (roles, permisos, datos sensibles): cada alumno ve solo su propia devolución.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el aviso de devolución se puede leer con lector de pantalla.
*   [ ] Otros: una devolución no confirmada se vuelve a pedir automáticamente; alerta si pasan 15 minutos.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Todos los que no ganaron recuperan todas sus monedas retenidas.
*   [ ] CA2: Si el Banco no confirma una devolución, se vuelve a pedir hasta que la confirme.
*   [ ] CA3: Cada alumno que no ganó recibe un aviso con el resultado.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Devolución de monedas al cerrar la subasta

**Escenario 1: devolución normal**

*   **Dado**: que oferté 850 y ganó otro alumno
*   **Cuando**: la subasta termina
*   **Entonces**: mis 850 monedas vuelven a estar disponibles y recibo "No ganaste la subasta, te devolvimos tus monedas"

**Escenario 2: el Banco no confirma**

*   **Dado**: que el Banco no confirmó mi devolución
*   **Cuando**: pasa el tiempo de reintento
*   **Entonces**: Mercado la vuelve a pedir hasta que se confirme

**Escenario 3: muchos participantes**

*   **Dado**: que 50 alumnos ofertaron
*   **Cuando**: la subasta termina
*   **Entonces**: las 49 devoluciones se piden en menos de 1 minuto

* * *

## Prototipo

*   **Mock API / Swagger**: eventos `HOLD_RELEASE_REQUESTED` / `HOLD_RELEASED`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08, Notificaciones.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: G08 (devolución de varias retenciones en un solo pedido).
*   Impacto en datos / migraciones: tabla `auction_pending_refund`.
*   Riesgos y mitigación (opcional): monedas que quedan retenidas por un error → reintento automático y alerta.


