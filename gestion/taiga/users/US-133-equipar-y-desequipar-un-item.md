# [G11 — Equipar y desequipar un ítem]

> **Taiga Ref:** #133 | **ID:** 9539931
> **Épica:** [#131 — G11 — Inventario, Equipamiento y Consumo del Alumno](../epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** decidir cuándo tener un ítem activo y cuándo guardarlo
*   **Para:** reservarlo para el desafío que de verdad me importa, en vez de gastarlo en cualquier ejercicio

Notas / Observaciones
---------------------

*   **Reglas de negocio:** desequipar no tiene costo. Solo puede haber **un ítem equipado por verbo de efecto**, para que nunca haya ambigüedad sobre cuál se consume ante un fallo.
*   **Validaciones:** solo se equipan ítems marcados como equipables. El ítem debe estar disponible y pertenecer al ALUMNO y al curso/cohorte.
*   **Datos obligatorios:** ID del ítem de inventario, ID del ALUMNO, curso/cohorte, verbo de efecto, estado.
*   **Performance (tiempos, volumen, límites):** operación puntual y liviana; sin exigencias particulares más allá de resolver bien la concurrencia.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO solo puede equipar/desequipar sus propios ítems.
*   **Accesibilidad (WCAG/teclado/lectores):** la acción de equipar debe ser alcanzable por teclado y el resultado anunciado a lectores de pantalla.
*   **Otros:** la garantía de "un ítem por verbo" se resuelve con una restricción de unicidad en la base de datos, no solo con una validación en el código, para cubrir el caso de dos pestañas equipando al mismo tiempo.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO puede equipar un ítem disponible, que pasa a estado equipado.
*   **CA2:** el ALUMNO puede desequipar un ítem equipado, que vuelve a disponible sin costo ni consumo.
*   **CA3:** si el ALUMNO intenta equipar dos ítems con el mismo verbo de efecto, el sistema permite solo uno y rechaza el segundo.
*   **Extras (opcional):** el inventario indica visualmente qué verbo de efecto ya está cubierto.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Gestión del estado equipado de los ítems del inventario

**Escenario 1**

*   **Dado:** un ítem disponible y equipable del ALUMNO
*   **Cuando:** lo equipa
*   **Entonces:** el ítem pasa a estado equipado

**Escenario 2**

*   **Dado:** un ítem equipado del ALUMNO
*   **Cuando:** lo desequipa
*   **Entonces:** el ítem vuelve a estado disponible sin costo ni consumo

**Escenario 3**

*   **Dado:** dos ítems disponibles del ALUMNO con el mismo verbo de efecto
*   **Cuando:** intenta equipar ambos
*   **Entonces:** el sistema permite equipar solo uno y rechaza el segundo

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `POST /api/market/inventory/{id}/equip`, `POST /api/market/inventory/{id}/unequip`

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

*   **Servicios involucrados:** Mercado (dueño).
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** agrega el estado "equipado" y un índice único filtrado por `(alumno, curso/cohorte, verbo de efecto)` para garantizar la unicidad.
*   **Riesgos y mitigación (opcional):** dos pestañas equipando a la vez es una carrera real. Mitigación: la restricción de unicidad en la base y una prueba de concurrencia obligatoria antes de cerrar la historia.

