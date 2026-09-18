# [G11 — Recuperar mis monedas si la compra no se completó]

> **Taiga Ref:** #141 | **ID:** 9539964
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** que me devuelvan mis monedas si finalmente no recibí el ítem
*   **Para:** confiar en el Mercado aunque el sistema falle

Notas / Observaciones
---------------------

*   **Reglas de negocio:** una orden compensada es un estado terminal — no se reintenta, el ALUMNO compra de nuevo si quiere el ítem. La devolución se acredita al mismo saldo simulado del que se descontó.
*   **Validaciones:** solo se compensa una orden que efectivamente cobró y no entregó.
*   **Datos obligatorios:** ID de la orden, clave de idempotencia de la devolución, estado, motivo.
*   **Performance (tiempos, volumen, límites):** los reintentos de devolución usan un esquema de espera creciente, para no saturar el propio servicio.
*   **Seguridad (roles, permisos, datos sensibles):** la compensación queda auditada con el motivo.
*   **Accesibilidad (WCAG/teclado/lectores):** el aviso al ALUMNO debe dejar claro en texto que la compra no tuvo costo.
*   **Otros:** si la devolución falla tras agotar los reintentos, la orden queda visible en un listado para intervención manual — nunca se descarta en silencio.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** si el cobro se completó pero el ítem no se pudo acreditar, al agotarse los reintentos el sistema devuelve las monedas y deja la orden en estado terminal.
*   **CA2:** una devolución que se reintenta no acredita monedas dos veces.
*   **CA3:** si la devolución tampoco se pudo completar, la orden queda marcada como pendiente de compensación y visible en un listado.
*   **Extras (opcional):** el ALUMNO recibe un aviso indicando que la compra no se completó y no tuvo costo.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Compensación de órdenes cobradas que no pudieron completarse

**Escenario 1**

*   **Dado:** que el cobro se completó pero el ítem no se pudo acreditar en el inventario
*   **Cuando:** se agotan los reintentos de entrega
*   **Entonces:** se devuelven las monedas al ALUMNO y la orden queda en estado terminal

**Escenario 2**

*   **Dado:** que ya se emitió la devolución de una orden
*   **Cuando:** el proceso de devolución se ejecuta de nuevo por un reintento
*   **Entonces:** no se acreditan monedas por segunda vez

**Escenario 3**

*   **Dado:** que la devolución no se pudo completar tras agotar los reintentos
*   **Cuando:** el proceso termina
*   **Entonces:** la orden queda marcada como pendiente de compensación en un listado de revisión

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** proceso interno de compensación sobre el saldo simulado (sin endpoint público propio en esta etapa)

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 8
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 8 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (orquesta y ejecuta la compensación sobre su propio saldo simulado).
*   **Módulos afectados:** Órdenes, Inventario.
*   **Otros equipos / aprobaciones:** ninguna bloqueante hoy.
*   **Impacto en datos / migraciones:** agrega estados de compensación a la orden y su clave de idempotencia de devolución.
*   **Riesgos y mitigación (opcional):** que esta historia se postergue por ser "el camino de error". Mitigación: no se puede cerrar "Comprar un ítem" sin los escenarios de falla de esta historia.

