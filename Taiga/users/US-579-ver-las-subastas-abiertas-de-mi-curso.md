# [G11 — Ver las subastas abiertas de mi curso]

> **Taiga Ref:** #579 | **ID:** 9549019
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Ver las subastas abiertas de mi curso

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO
*   **Quiero**: ver las subastas de mi curso con el ítem, la oferta más alta y el tiempo que falta
*   **Para**: decidir si me conviene participar

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: solo se muestran subastas abiertas o próximas del curso del alumno. Cada una indica si el alumno va ganando, si lo superaron o si todavía no participó.
*   [ ] Validaciones: el alumno debe estar matriculado en el curso.
*   [ ] Datos obligatorios: nombre e imagen del ítem, oferta más alta, tiempo restante, estado del alumno.
*   [ ] Performance (tiempos, volumen, límites): la lista carga en menos de 1 segundo; el tiempo restante se calcula con la hora del servidor.
*   [ ] Seguridad (roles, permisos, datos sensibles): no se muestra quién hizo cada oferta, solo el monto.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el tiempo restante se lee también como texto ("faltan 2 horas").
*   [ ] Otros: funciona en celular (RF-NFR-06); no aparece el aviso de "esta sección requiere una computadora".

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Solo aparecen subastas del curso del alumno.
*   [ ] CA2: La pantalla funciona desde el celular.
*   [ ] CA3: Cada subasta muestra si el alumno va ganando, lo superaron o no participa.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Listado de subastas para el alumno

**Escenario 1: listado del curso**

*   **Dado**: que estoy en "Programación IV 2026", hay 2 subastas abiertas en mi curso y 1 en otro curso
*   **Cuando**: entro a Subastas
*   **Entonces**: veo solo las 2 de mi curso, con la oferta más alta y el tiempo que falta

**Escenario 2: desde el celular**

*   **Dado**: que entro desde el celular
*   **Cuando**: abro Subastas
*   **Entonces**: la pantalla se ve y funciona normalmente

**Escenario 3: sin subastas**

*   **Dado**: que no hay subastas abiertas en mi curso
*   **Cuando**: entro a Subastas
*   **Entonces**: veo el mensaje "No hay subastas activas por ahora"

* * *

## Prototipo

*   **Mock API / Swagger**: `GET /api/v1/market/auctions?courseId={id}`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 2
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Cursos y Matrícula G02.
*   Módulos afectados: Subastas (front web y móvil).
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: ninguno (solo lectura).
*   Riesgos y mitigación (opcional): diferencias de hora entre el celular y el servidor → el tiempo restante siempre se calcula con la hora del servidor.


