# [G11 — Lanzar un ítem a subasta]

> **Taiga Ref:** #578 | **ID:** 9549018
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Lanzar un ítem a subasta

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: publicar un ítem de equipamiento en subasta, eligiendo cuándo empieza, cuánto dura y, si quiero, una oferta mínima
*   **Para**: generar una dinámica de competencia en mi curso

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: solo se subasta equipamiento activo en el catálogo del curso; las vidas no se subastan. Se puede guardar como borrador o dejarla programada. Cuando llega la hora de inicio se abre sola y se avisa a los alumnos (RF-NOT-02). La subasta guarda el ítem tal como estaba al momento de crearla.
*   [ ] Validaciones: duración entre 1 hora y 7 días; oferta mínima de 1 moneda o más (opcional); la fecha de inicio no puede estar en el pasado.
*   [ ] Datos obligatorios: curso, ítem, fecha y hora de inicio, duración.
*   [ ] Performance (tiempos, volumen, límites): la subasta se abre como máximo 1 minuto después de su hora de inicio.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo un PROFESOR del curso.
*   [ ] Accesibilidad (WCAG/teclado/lectores): formulario completo con teclado; selector de fecha con etiqueta legible por lector de pantalla.
*   [ ] Otros: queda registrado quién la creó y cuándo.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Un profesor del curso puede crear una subasta y queda programada con los datos cargados.
*   [ ] CA2: El sistema no permite subastar una vida ni un ítem inactivo.
*   [ ] CA3: Al llegar la hora de inicio, la subasta aparece como abierta y los alumnos reciben un aviso.
*   [ ] Extras (opcional): Un alumno que intenta crear una subasta recibe un error de permisos.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Lanzamiento de subastas por el profesor

**Escenario 1: lanzamiento correcto**

*   **Dado**: que soy profesor del curso "Programación IV 2026" y el ítem "Escudo Baluarte" está activo en el catálogo
*   **Cuando**: creo una subasta de 48 horas con oferta mínima de 800 monedas
*   **Entonces**: la subasta queda programada y, al llegar la hora de inicio, se abre y los alumnos reciben un aviso

**Escenario 2: intento de subastar una vida**

*   **Dado**: que elijo la "Poción de Vida" como ítem
*   **Cuando**: intento crear la subasta
*   **Entonces**: el sistema no lo permite y muestra "Las vidas no se pueden subastar"

**Escenario 3: usuario sin permiso**

*   **Dado**: que soy alumno del curso
*   **Cuando**: intento crear una subasta
*   **Entonces**: el sistema me informa que no tengo permiso

* * *

## Prototipo

*   **Mock API / Swagger**: `POST /api/v1/market/auctions`, `GET /api/v1/market/auctions/{id}`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Identidad G01, Notificaciones.
*   Módulos afectados: Subastas, Catálogo.
*   Otros equipos / aprobaciones: PO (confirmar que las vidas no se subastan).
*   Impacto en datos / migraciones: tabla `auction`.
*   Riesgos y mitigación (opcional): que un profesor cree subastas muy largas y deje monedas retenidas mucho tiempo → duración máxima de 7 días.


