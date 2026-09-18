# [G11 — Modificar un ítem]

> **Taiga Ref:** #95 | **ID:** 9535768
> **Épica:** [#90 — G11 — Gestión del Catálogo de Mercado](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ADMIN
*   **Quiero:** modificar la información de un ítem existente
*   **Para:** mantener actualizado el catálogo

Notas / Observaciones
---------------------

*   **Reglas de negocio:** la modificación debe respetar las categorías y reglas definidas para el Mercado.
*   **Validaciones:** el ítem debe existir y quien modifica debe tener rol ADMIN.
*   **Datos obligatorios:** ID del ítem y campos que se desean modificar (nombre, tipo, precio, imagen, descripción corta, estado).
*   **Performance (tiempos, volumen, límites):** la actualización debe realizarse de forma transaccional.
*   **Seguridad (roles, permisos, datos sensibles):** solo ADMIN puede modificar información del catálogo.
*   **Accesibilidad (WCAG/teclado/lectores):** el formulario debe ser accesible.
*   **Otros:** debe evitarse modificar accidentalmente un ítem perteneciente a otro curso/cohorte.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** ADMIN puede modificar un ítem existente.
*   **CA2:** el sistema rechaza la modificación de un ítem inexistente.
*   **CA3:** el sistema impide que un rol distinto de ADMIN modifique información del catálogo.
*   **Extras (opcional):** la modificación queda registrada para permitir trazabilidad.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Modificación de ítems del Mercado

**Escenario 1**

*   **Dado:** un ítem existente y un ADMIN autenticado
*   **Cuando:** modifica uno de sus datos válidos y confirma
*   **Entonces:** el sistema actualiza la información del ítem

**Escenario 2**

*   **Dado:** un ADMIN autenticado
*   **Cuando:** intenta modificar un ítem inexistente
*   **Entonces:** el sistema informa que el ítem no existe

**Escenario 3**

*   **Dado:** un ALUMNO o PROFESOR autenticado (sin permisos de ADMIN)
*   **Cuando:** intenta modificar un ítem
*   **Entonces:** el sistema rechaza la operación

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `PUT/PATCH /api/market/items/{id}`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 3
*   **Prioridad (MoSCoW / Numérica):** Should / 2

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 3 | Debería / 2 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado, Identidad (validación de rol ADMIN).
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** actualización de registros existentes.
*   **Riesgos y mitigación (opcional):** cambios incorrectos de precio o tipo. Mitigación: validar datos antes de persistir.

