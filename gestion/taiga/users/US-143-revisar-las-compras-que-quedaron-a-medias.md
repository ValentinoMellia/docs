# [G11 — Revisar las compras que quedaron a medias]

> **Taiga Ref:** #143 | **ID:** 9539966
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** PROFESOR
*   **Quiero:** ver las compras de mi curso que no se pudieron completar ni devolver
*   **Para:** poder responderle a un ALUMNO que reclama que pagó y no recibió nada

Notas / Observaciones
---------------------

*   **Reglas de negocio:** solo se listan órdenes en estado pendiente de compensación; es una vista de excepción, no un listado general de compras.
*   **Validaciones:** el PROFESOR solo ve las órdenes de su propio curso (RF-USR-08). Mientras Cursos/Matrícula no exista, se simula un curso fijo asociado al PROFESOR de prueba.
*   **Datos obligatorios:** ALUMNO, ítem, monto, fecha, estado, último paso que falló.
*   **Performance (tiempos, volumen, límites):** volumen bajo por definición; si el listado crece, es señal de un problema sistémico a atender.
*   **Seguridad (roles, permisos, datos sensibles):** rol PROFESOR con alcance a su curso; no se expone detalle técnico interno.
*   **Accesibilidad (WCAG/teclado/lectores):** tabla con encabezados asociados, navegable por teclado.
*   **Otros:** el mismo dato podría consumirlo en el futuro un panel de Backoffice; conviene resolver la consulta pensando en los dos alcances desde ahora.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el PROFESOR puede ver, en la administración de su curso, las órdenes pendientes de compensación con ALUMNO, ítem, monto y fecha.
*   **CA2:** el PROFESOR no ve órdenes de un curso que no administra.
*   **CA3:** el detalle de una orden pendiente indica en qué paso quedó, sin exponer trazas ni mensajes técnicos internos.
*   **Extras (opcional):** el listado permite exportar los casos del período.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Listado de órdenes pendientes de compensación por curso

**Escenario 1**

*   **Dado:** que en su curso hay órdenes pendientes de compensación
*   **Cuando:** el PROFESOR abre el listado de excepciones
*   **Entonces:** ve las órdenes con ALUMNO, ítem, monto y fecha

**Escenario 2**

*   **Dado:** que existe una orden pendiente en un curso que no administra
*   **Cuando:** el PROFESOR abre su listado
*   **Entonces:** esa orden no aparece

**Escenario 3**

*   **Dado:** que el PROFESOR abre el detalle de una orden pendiente
*   **Cuando:** el sistema muestra la información
*   **Entonces:** indica en qué paso del proceso quedó, sin exponer trazas técnicas internas

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `GET /api/market/orders?status=PENDING_COMPENSATION&courseCohortId={id}`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 3
*   **Prioridad (MoSCoW / Numérica):** Could / 3

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 3 | Podría / 3 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado. Cursos/Matrícula — mockeado, para verificar la propiedad del curso.
*   **Módulos afectados:** Órdenes.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** ninguno adicional; requiere índice por estado sobre la tabla de órdenes.
*   **Riesgos y mitigación (opcional):** que el listado exista pero nadie lo revise. Mitigación: la alerta de observabilidad definida a nivel épico.

