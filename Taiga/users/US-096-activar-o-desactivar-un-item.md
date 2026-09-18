# [G11 — Activar o desactivar un ítem]

> **Taiga Ref:** #96 | **ID:** 9535772
> **Épica:** [#90 — G11 — Gestión del Catálogo de Mercado](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ADMIN
*   **Quiero:** activar o desactivar un ítem del catálogo
*   **Para:** controlar qué ítems se encuentran disponibles para los alumnos

Notas / Observaciones
---------------------

*   **Reglas de negocio:** un ítem que no se encuentre disponible no debe poder seleccionarse para una operación de compra.
*   **Validaciones:** el ítem debe existir y quien modifica el estado debe tener rol ADMIN.
*   **Datos obligatorios:** ID del ítem, estado y fecha/hora de modificación.
*   **Performance (tiempos, volumen, límites):** el cambio de estado debe persistirse correctamente antes de confirmar la operación.
*   **Seguridad (roles, permisos, datos sensibles):** solo ADMIN puede modificar el estado.
*   **Accesibilidad (WCAG/teclado/lectores):** el estado debe ser identificable de forma textual y no depender únicamente del color.
*   **Otros:** desactivar un ítem no implica eliminarlo físicamente (RF-NFR-01 — baja lógica en toda la plataforma).

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** ADMIN puede desactivar un ítem disponible.
*   **CA2:** un ítem desactivado no aparece como disponible para el ALUMNO.
*   **CA3:** ADMIN puede volver a activar un ítem desactivado.
*   **Extras (opcional):** el sistema registra el cambio de estado para mantener trazabilidad.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Activación y desactivación de ítems

**Escenario 1**

*   **Dado:** un ítem activo en el catálogo
*   **Cuando:** un ADMIN lo desactiva
*   **Entonces:** el sistema cambia su estado a inactivo y deja de mostrarlo como disponible

**Escenario 2**

*   **Dado:** un ítem inactivo
*   **Cuando:** un ADMIN lo activa
*   **Entonces:** el sistema cambia su estado a activo

**Escenario 3**

*   **Dado:** un ALUMNO o PROFESOR autenticado (sin permisos de ADMIN)
*   **Cuando:** intenta cambiar el estado de un ítem
*   **Entonces:** el sistema rechaza la operación

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `PATCH /api/market/items/{id}/status`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 3
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 3 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado, Identidad (validación de rol ADMIN).
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** se modifica el estado del registro del ítem.
*   **Riesgos y mitigación (opcional):** que un ítem desactivado continúe disponible por caché o consultas desactualizadas. Mitigación: validar el estado en backend.

