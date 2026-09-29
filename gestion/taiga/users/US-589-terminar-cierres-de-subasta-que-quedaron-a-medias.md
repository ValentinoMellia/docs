# [G11 — Terminar cierres de subasta que quedaron a medias]

> **Taiga Ref:** #589 | **ID:** 9549029
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Terminar cierres de subasta que quedaron a medias

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO que participa en subastas
*   **Quiero**: que si algo falla durante el cierre, el sistema lo termine solo
*   **Para**: no quedarme sin mi ítem ni con monedas retenidas

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: cuando Mercado arranca, busca subastas vencidas que no terminaron de cerrarse y retoma el cierre desde donde quedó, sin repetir cobros ni entregas. Los mensajes al Banco se guardan antes de enviarse, así no se pierden si la mensajería (Kafka) está caída. Si el Banco no responde, se reintenta cada vez con más espera.
*   [ ] Validaciones: una subasta no se puede entregar ni cobrar dos veces.
*   [ ] Datos obligatorios: estado del cierre, cantidad de reintentos.
*   [ ] Performance (tiempos, volumen, límites): alerta si un cierre tarda más de 2 minutos.
*   [ ] Seguridad (roles, permisos, datos sensibles): proceso interno, sin acceso de usuarios.
*   [ ] Accesibilidad (WCAG/teclado/lectores): no aplica.
*   [ ] Otros: mientras el cierre está pendiente, las monedas del ganador siguen retenidas.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Si Mercado se reinicia durante un cierre, la subasta se termina correctamente una sola vez.
*   [ ] CA2: Si Kafka no está disponible, los mensajes se envían cuando vuelve.
*   [ ] CA3: Si el Banco no responde, la subasta queda "pendiente de cobro", se reintenta y se dispara una alerta.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Recuperación de cierres incompletos

**Escenario 1: reinicio de Mercado**

*   **Dado**: que Mercado se reinició después de elegir al ganador
*   **Cuando**: vuelve a arrancar
*   **Entonces**: retoma el cierre y lo completa sin cobrar dos veces

**Escenario 2: Banco caído**

*   **Dado**: que el Banco no responde al pedido de cobro
*   **Cuando**: se agotan los primeros reintentos
*   **Entonces**: la subasta queda "pendiente de cobro", las monedas siguen retenidas y se dispara una alerta

**Escenario 3: mensajería caída**

*   **Dado**: que Kafka no está disponible durante el cierre
*   **Cuando**: vuelve a estar disponible
*   **Entonces**: los mensajes guardados se envían y el cierre continúa

* * *

## Prototipo

*   **Mock API / Swagger**: no aplica (proceso interno)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 5
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: tabla `outbox_event` (mensajes guardados antes de enviarse); estado "pendiente de cobro" en `auction`.
*   Riesgos y mitigación (opcional): reintentos infinitos → tope de reintentos y alerta para revisión manual.


