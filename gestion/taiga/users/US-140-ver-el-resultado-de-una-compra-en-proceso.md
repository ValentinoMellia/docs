# [G11 — Ver el resultado de una compra en proceso]

> **Taiga Ref:** #140 | **ID:** 9539963
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** enterarme del resultado de mi compra aunque no sea inmediato
*   **Para:** saber si tengo el ítem, sin quedarme mirando una pantalla trabada

Notas / Observaciones
---------------------

*   **Reglas de negocio:** la respuesta inicial no lleva el resultado del cobro. La orden atraviesa estados intermedios hasta llegar a uno terminal.
*   **Validaciones:** el ALUMNO solo consulta el estado de sus propias órdenes.
*   **Datos obligatorios:** ID de la orden, estado, fecha de creación, precio aplicado, resultado final.
*   **Performance (tiempos, volumen, límites):** si la consulta de estado es por sondeo, definir una frecuencia que no sobrecargue el servicio.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO solo accede a sus propias órdenes (RF-USR-07).
*   **Accesibilidad (WCAG/teclado/lectores):** el cambio de estado debe anunciarse a lectores de pantalla mediante una región activa.
*   **Otros:** un indicador que dice "en proceso" es correcto; un spinner infinito sin explicación, no.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** al confirmar la compra, el sistema responde de inmediato indicando que está en proceso.
*   **CA2:** la pantalla llega a mostrar el resultado final sin que el ALUMNO tenga que recargar.
*   **CA3:** si el ALUMNO cierra la pantalla mientras la compra está en proceso, al volver encuentra el resultado final en su listado de órdenes.
*   **Extras (opcional):** el listado de órdenes muestra el motivo cuando el resultado fue un rechazo o una devolución.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Seguimiento del estado de una orden de compra hasta su resultado final

**Escenario 1**

*   **Dado:** que el ALUMNO confirma la compra
*   **Cuando:** el sistema recibe la petición
*   **Entonces:** responde de inmediato indicando que la compra está en proceso

**Escenario 2**

*   **Dado:** que la compra está en proceso
*   **Cuando:** la operación llega a un estado terminal
*   **Entonces:** la pantalla muestra el resultado final sin necesidad de recargar

**Escenario 3**

*   **Dado:** que el ALUMNO cerró la pestaña con la compra en proceso
*   **Cuando:** vuelve a entrar a la plataforma
*   **Entonces:** encuentra el resultado final de esa compra en su listado de órdenes

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `GET /api/market/orders/{orderId}`, `GET /api/market/orders`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Should / 2

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Debería / 2 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado.
*   **Módulos afectados:** Órdenes.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** ninguno adicional; consulta sobre la tabla de órdenes.
*   **Riesgos y mitigación (opcional):** un sondeo demasiado frecuente puede degradar el servicio. Mitigación: definir la frecuencia como parámetro configurable.

