# [G11 — Consultar detalle de un artículo]

> **Taiga Ref:** #98 | **ID:** 9535782
> **Épica:** [#90 — G11 — Catálogo Abierto por Plantillas](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** consultar el detalle de un artículo del Mercado
*   **Para:** conocer sus características y condiciones antes de adquirirlo

Notas / Observaciones
---------------------

*   **Reglas de negocio:** el artículo debe pertenecer al contexto de Mercado correspondiente al curso del ALUMNO.
*   **Validaciones:** el artículo debe existir y encontrarse disponible para consulta.
*   **Datos obligatorios:** ID del artículo, nombre, tipo, precio, imagen, descripción y estado.
*   **Performance (tiempos, volumen, límites):** la información del detalle debe cargarse dentro de tiempos adecuados para una consulta interactiva.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO solo puede consultar elementos disponibles en su contexto de curso.
*   **Accesibilidad (WCAG/teclado/lectores):** los datos deben contar con etiquetas y estructura accesible.
*   **Otros:** el detalle debe permitir distinguir entre vidas y equipamiento. Esta historia queda con prioridad reducida respecto de las demás porque el prototipo actual ya muestra esta información integrada en la tarjeta del catálogo (#92); se conserva documentada para cuando el catálogo crezca y una tarjeta no alcance para mostrar todo.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO puede seleccionar un artículo del catálogo.
*   **CA2:** el sistema muestra la información definida del elemento seleccionado.
*   **CA3:** el sistema informa cuando el elemento solicitado no existe o no está disponible.
*   **Extras (opcional):** el detalle indica claramente el precio del artículo.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Consulta del detalle de un artículo

**Escenario 1**

*   **Dado:** un artículo disponible en el catálogo
*   **Cuando:** el ALUMNO selecciona el artículo
*   **Entonces:** el sistema muestra su información detallada

**Escenario 2**

*   **Dado:** un ítem que pertenece a otro curso
*   **Cuando:** el ALUMNO intenta consultar su detalle
*   **Entonces:** el sistema rechaza el acceso al artículo

**Escenario 3**

*   **Dado:** un identificador de artículo inexistente
*   **Cuando:** el ALUMNO solicita su detalle
*   **Entonces:** el sistema informa que el artículo no existe

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `GET /api/market/items/{id}`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 2
*   **Prioridad (MoSCoW / Numérica):** Should / 2

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 2 | Debería / 2 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado, Identidad y Cursos/Matrícula — mockeados mientras esos equipos no tengan nada disponible.
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** Equipo de Cursos y Matrícula (cuando exista).
*   **Impacto en datos / migraciones:** sin migraciones previstas.
*   **Riesgos y mitigación (opcional):** validar el contexto del curso antes de devolver información, incluso contra el valor simulado.

