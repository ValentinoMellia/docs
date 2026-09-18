# [G11 — Consultar los ítems vencidos del curso]

> **Taiga Ref:** #788 | **ID:** 9552680
> **Épica:** [#770 — G11 — Vencimiento de Ítems del Inventario](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Consultar los ítems vencidos del curso

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: ver cuántos ítems vencieron sin usarse en mi curso
*   **Para**: saber si los vencimientos que puse son razonables

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: Mercado entrega por curso la cantidad de ítems vencidos sin usar, agrupados por ítem, y el total de monedas que representaban. No muestra nombres de alumnos.
*   [ ] Validaciones: se puede filtrar por rango de fechas.
*   [ ] Datos obligatorios: curso, rango de fechas.
*   [ ] Performance (tiempos, volumen, límites): responde en menos de 1 segundo.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo el PROFESOR del curso y el ADMIN.
*   [ ] Accesibilidad (WCAG/teclado/lectores): la pantalla la arma Backoffice o Mercado; los datos se entregan como texto y números.
*   [ ] Otros: puede sumarse al reporte de ventas del curso.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: El profesor ve la cantidad de ítems vencidos por ítem en su curso.
*   [ ] CA2: Se puede filtrar por fechas.
*   [ ] CA3: Un profesor no puede ver los datos de un curso ajeno.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Reporte de ítems vencidos

**Escenario 1: reporte del curso**

*   **Dado**: que en mi curso vencieron 12 escudos sin usar
*   **Cuando**: consulto los ítems vencidos
*   **Entonces**: veo "Escudo Reforzado: 12" y las monedas que representaban

**Escenario 2: filtro por fechas**

*   **Dado**: que quiero ver solo octubre
*   **Cuando**: filtro del 1 al 31 de octubre
*   **Entonces**: veo solo los vencidos en ese período

**Escenario 3: curso ajeno**

*   **Dado**: que soy profesor de otro curso
*   **Cuando**: pido el reporte de "Programación IV 2026"
*   **Entonces**: el sistema me informa que no tengo permiso

* * *

## Prototipo

*   **Mock API / Swagger**: `GET /api/v1/market/admin/expired-items?courseId={id}&from={fecha}&to={fecha}`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 2
*   **Prioridad (MoSCoW / Numérica)**: Could

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Backoffice G12.
*   Módulos afectados: Reportes.
*   Otros equipos / aprobaciones: G12 (si lo muestran en su panel).
*   Impacto en datos / migraciones: ninguno.
*   Riesgos y mitigación (opcional): —


