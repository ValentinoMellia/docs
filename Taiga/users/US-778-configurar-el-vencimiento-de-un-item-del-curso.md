# [G11 — Configurar el vencimiento de un ítem del curso]

> **Taiga Ref:** #778 | **ID:** 9552674
> **Épica:** [#770 — G11 — Vencimiento de Ítems del Inventario](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Configurar el vencimiento de un ítem del curso

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: indicar que un ítem de mi curso vence en una fecha o a los días de recibirlo
*   **Para**: que los alumnos usen los ítems en el momento del cursado que me interesa

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: el vencimiento es opcional. Puede ser una fecha fija o una cantidad de días desde que el alumno recibe el ítem. No se puede poner vencimiento a las vidas. Un cambio de vencimiento vale solo para los ítems que se entreguen después del cambio (RF-CFG-06).
*   [ ] Validaciones: la fecha fija no puede estar en el pasado; la cantidad de días va de 1 a 180.
*   [ ] Datos obligatorios: ítem del curso, tipo de vencimiento (fecha fija o días) y su valor.
*   [ ] Performance (tiempos, volumen, límites): no aplica.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo el PROFESOR del curso; el ADMIN puede verlo pero no cambia la curaduría del curso.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el selector de fecha se puede usar con teclado y tiene etiqueta para lector de pantalla.
*   [ ] Otros: queda registrado quién hizo el cambio y cuándo.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: El profesor puede poner, cambiar o quitar el vencimiento de un ítem de su curso.
*   [ ] CA2: El sistema rechaza fechas pasadas, cantidades de días fuera de rango y vencimientos en vidas.
*   [ ] CA3: Cambiar el vencimiento no modifica los ítems que los alumnos ya tienen.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Configuración de vencimiento por el profesor

**Escenario 1: vencimiento por fecha**

*   **Dado**: que soy profesor de "Programación IV 2026"
*   **Cuando**: le pongo al "Escudo Reforzado" vencimiento el 30 de octubre
*   **Entonces**: los alumnos que lo compren ven que vence el 30 de octubre

**Escenario 2: fecha inválida**

*   **Dado**: que hoy es 16 de septiembre
*   **Cuando**: intento poner vencimiento el 1 de septiembre
*   **Entonces**: el sistema me avisa que la fecha no puede estar en el pasado

**Escenario 3: cambio posterior**

*   **Dado**: que un alumno ya tiene un escudo que vence en 10 días
*   **Cuando**: cambio el vencimiento del ítem a 5 días
*   **Entonces**: el escudo del alumno sigue venciendo en 10 días

* * *

## Prototipo

*   **Mock API / Swagger**: `PUT /api/v1/market/courses/{courseId}/offers/{offerId}` (campos `expirationType` y `expirationValue`)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Identidad G01.
*   Módulos afectados: Catálogo.
*   Otros equipos / aprobaciones: PO (confirmar los dos tipos de vencimiento).
*   Impacto en datos / migraciones: campos `expiration_type` y `expiration_value` en la oferta del catálogo.
*   Riesgos y mitigación (opcional): —


