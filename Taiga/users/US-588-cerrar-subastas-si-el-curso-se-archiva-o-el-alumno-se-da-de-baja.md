# [G11 — Cerrar subastas si el curso se archiva o el alumno se da de baja]

> **Taiga Ref:** #588 | **ID:** 9549028
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Cerrar subastas si el curso se archiva o el alumno se da de baja

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR del curso
*   **Quiero**: que las subastas se ajusten solas cuando el curso se archiva o un alumno deja el curso
*   **Para**: que no queden monedas retenidas ni ganadores que ya no están en el curso

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio (curso archivado): se cancelan sus subastas abiertas, se devuelven todas las monedas y no se pueden crear nuevas (RF-CUR-09).
*   [ ] Reglas de negocio (alumno dado de baja): se le devuelven las monedas de sus ofertas y esas ofertas dejan de contar. Si iba ganando, pasa a ganar la siguiente oferta más alta. Es la única vez que una oferta se retira, y la retira el sistema, no el alumno.
*   [ ] Validaciones: cada aviso de Cursos se procesa una sola vez.
*   [ ] Datos obligatorios: curso, alumno (en caso de baja), motivo.
*   [ ] Performance (tiempos, volumen, límites): se procesa en menos de 1 minuto desde el aviso.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo se aceptan avisos del servicio de Cursos.
*   [ ] Accesibilidad (WCAG/teclado/lectores): no aplica.
*   [ ] Otros: se avisa a los participantes afectados.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Después de archivar un curso, no quedan subastas abiertas ni monedas retenidas en ese curso.
*   [ ] CA2: Un alumno dado de baja no puede ganar una subasta.
*   [ ] CA3: Si el aviso llega repetido, no se hace nada dos veces.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Subastas y ciclo de vida del curso

**Escenario 1: curso archivado**

*   **Dado**: que el curso tiene una subasta abierta con ofertas
*   **Cuando**: el curso se archiva
*   **Entonces**: la subasta se cancela y se devuelven todas las monedas

**Escenario 2: se da de baja el alumno que iba ganando**

*   **Dado**: que el alumno que iba ganando se da de baja del curso
*   **Cuando**: llega el aviso de baja
*   **Entonces**: se le devuelven sus monedas y pasa a ganar la siguiente oferta más alta

**Escenario 3: intento de crear subasta en curso archivado**

*   **Dado**: que el curso está archivado
*   **Cuando**: el profesor intenta crear una subasta
*   **Entonces**: el sistema informa que el curso está en solo lectura

* * *

## Prototipo

*   **Mock API / Swagger**: eventos `CURSO_ARCHIVADO` y `ALUMNO_DESMATRICULADO` (tópico `cursos.ciclo-vida`)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Cursos y Matrícula G02, Banco G08.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: G02 y PO (confirmar que archivar cancela las subastas).
*   Impacto en datos / migraciones: nuevo estado de oferta "retirada por el sistema".
*   Riesgos y mitigación (opcional): —


