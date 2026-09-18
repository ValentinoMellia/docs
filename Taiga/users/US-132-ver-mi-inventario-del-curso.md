# [G11 — Ver mi inventario del curso]

> **Taiga Ref:** #132 | **ID:** 9539908
> **Épica:** [#131 — G11 — Inventario, Equipamiento y Consumo del Alumno](../epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** ver los ítems que tengo en mi curso y en qué estado está cada uno
*   **Para:** saber con qué cuento antes de encarar un desafío y poder decidir qué equipar

Notas / Observaciones
---------------------

*   **Reglas de negocio:** cada ítem es una instancia individual, no un contador (RF-REC-05). Las recompensas de un curso solo se usan en ese curso (RF-REC-01). Un ítem consumido nunca vuelve a estar disponible.
*   **Validaciones:** el ALUMNO debe pertenecer al curso/cohorte consultado. Mientras Identidad y Cursos/Matrícula no estén disponibles, se simula un ALUMNO y un curso/cohorte fijos.
*   **Datos obligatorios:** ID del curso/cohorte, ID del ALUMNO, ID del ítem, nombre, tipo, estado (disponible/equipado/consumido), origen y fecha de consumo (si corresponde).
*   **Performance (tiempos, volumen, límites):** la consulta debe responder sin demoras perceptibles con el volumen esperado de ítems por alumno (decenas de instancias).
*   **Seguridad (roles, permisos, datos sensibles):** un ALUMNO nunca accede al inventario de otro ALUMNO (RF-USR-07).
*   **Accesibilidad (WCAG/teclado/lectores):** la tabla de inventario debe ser navegable por teclado; el estado del ítem no puede comunicarse solo por color.
*   **Otros:** cada ítem se muestra como fila individual (instancia), nunca agrupado tipo "Escudo x2".

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO puede ver los ítems de su curso/cohorte con su estado actual.
*   **CA2:** el ALUMNO no ve ítems de otro curso/cohorte.
*   **CA3:** un ítem consumido sigue apareciendo en el listado con su fecha de consumo, no desaparece.
*   **Extras (opcional):** los ítems se ordenan mostrando primero los disponibles.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Consulta del inventario de ítems del alumno

**Escenario 1**

*   **Dado:** un ALUMNO con ítems en su curso/cohorte
*   **Cuando:** consulta su inventario
*   **Entonces:** el sistema muestra una fila por instancia con su estado actual

**Escenario 2**

*   **Dado:** un ALUMNO con ítems en más de un curso/cohorte
*   **Cuando:** consulta el inventario de uno de ellos
*   **Entonces:** el sistema excluye los ítems de los otros cursos

**Escenario 3**

*   **Dado:** un ítem ya consumido por el ALUMNO
*   **Cuando:** consulta su inventario
*   **Entonces:** el ítem sigue visible con su estado "consumido" y su fecha, sin desaparecer del listado

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `GET /api/market/inventory?courseCohortId={id}`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado. Identidad y Cursos/Matrícula — mockeados por ahora.
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** crea la tabla de inventario con índices por curso/cohorte y por alumno.
*   **Riesgos y mitigación (opcional):** modelarlo como contador en vez de instancia individual haría imposible cumplir RF-REC-05 y obligaría a migrar datos después. Mitigación: verificarlo en la revisión del modelo antes de programar.

## Tareas Técnicas Asociadas en Taiga (7)

| Ref | Tarea | Estado | Alcance / Descripción |
| :---: | :--- | :---: | :--- |
| **#1112** | G11 - US-132 - T01 - Modelar entidad e índices de inventario con aislamiento por curso y alumno | New | Diseñar tabla student_inventory y entidad ItemInventario con índices por (student_id, course_id, state) e instancias individuales (RF-REC-05). |
| **#1113** | G11 - US-132 - T02 - Implementar endpoint REST de consulta de inventario por curso-cohorte | New | GET /api/v1/courses/{courseId}/inventory con extracción de X-User-Id, DTO de ítems, estado y ordenamiento de disponibles primero. |
| **#1114** | G11 - US-132 - T03 - Validar matriculación activa del alumno en el curso (contrato con Cursos G02) | New | Verificar pertenencia vía Gateway con Cursos y Matrícula (403 si no pertenece) garantizando aislamiento total de ítems por curso. |
| **#1115** | G11 - US-132 - T04 - Integrar pestaña Mercado/Inventario en la navegación contextual del curso (estilo Classroom) | New | Integrar en layout de curso (/courses/:courseId) con pestañas tipo Classroom (Novedades, Desafíos, Mercado/Inventario) y breadcrumbs. |
| **#1116** | G11 - US-132 - T05 - Maquetar componente de mochila: cards por instancia de ítem, badges de estado y cargas | New | Cards individuales por instancia, badges Disponible (verde), Equipado (azul), Consumido (gris con fecha), indicador de cargas y modal de detalle. |
| **#1117** | G11 - US-132 - T06 - Implementar filtros por estado, ordenamiento, empty state y accesibilidad WCAG | New | Filtros rápidos (Todos, Disponibles, Equipados, Consumidos), empty state amigable de curso sin ítems, foco visible y soporte ARIA. |
| **#1118** | G11 - US-132 - T07 - Desarrollar tests unitarios, de integración y pruebas de aislamiento entre cursos | New | Tests unitarios, validación de aislamiento multitenant entre cursos, persistencia de consumidos y cobertura de los 3 escenarios BDD. |
