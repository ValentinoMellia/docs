# [G11 — Activar un consumible con vigencia temporal]

> **Taiga Ref:** #134 | **ID:** 9539944
> **Épica:** [#131 — G11 — Inventario, Equipamiento y Consumo del Alumno](../epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** activar un consumible de duración limitada cuando yo decida
*   **Para:** aprovechar su efecto durante la sesión de estudio que estoy por empezar, y no que se active solo

Notas / Observaciones
---------------------

*   **Reglas de negocio:** hay ítems de **consumo puntual** (escudos: se gastan en el momento de un fallo) y de **vigencia temporal** (pociones/elixires: quedan activos durante una ventana de tiempo). Los de vigencia se **activan**, no se equipan, y no ocupan el lugar de equipamiento de su verbo.
*   **Validaciones:** el ítem debe estar disponible y ser de tipo vigencia. No se puede activar dos veces el mismo efecto en paralelo.
*   **Datos obligatorios:** ID del ítem, fecha/hora de activación, fecha/hora de vencimiento, verbo de efecto.
*   **Performance (tiempos, volumen, límites):** el cálculo de vigencia se resuelve comparando marcas de tiempo, sin proceso programado recorriendo inventarios.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO solo puede activar sus propios ítems.
*   **Accesibilidad (WCAG/teclado/lectores):** el tiempo restante debe estar en texto, no solo en una barra visual.
*   **Otros:** un ítem vencido no se reembolsa; el costo de oportunidad de decidir cuándo activarlo es parte del diseño.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO puede activar un consumible de vigencia disponible, que queda activo con una ventana temporal definida.
*   **CA2:** un consumible activado no ocupa el lugar de equipamiento de su verbo de efecto.
*   **CA3:** al vencer la ventana, el ítem se marca como vencido y no se reembolsa.
*   **Extras (opcional):** el inventario muestra el tiempo restante de cada efecto activo.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Activación y vencimiento de consumibles con vigencia temporal

**Escenario 1**

*   **Dado:** una poción disponible en el inventario del ALUMNO
*   **Cuando:** la activa
*   **Entonces:** queda en estado activo con una fecha y hora de vencimiento

**Escenario 2**

*   **Dado:** una poción activa del ALUMNO
*   **Cuando:** se consulta su inventario
*   **Entonces:** el ítem no ocupa el lugar de equipamiento de su verbo

**Escenario 3**

*   **Dado:** que la ventana de vigencia de la poción del ALUMNO se agotó
*   **Cuando:** consulta su inventario
*   **Entonces:** la poción figura como vencida y no se le devuelven monedas por ella

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `POST /api/market/inventory/{id}/activate`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 8
*   **Prioridad (MoSCoW / Numérica):** Could / 3

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 8 | Podría / 3 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño).
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** agrega fecha de activación y de vencimiento al ítem de inventario.
*   **Riesgos y mitigación (opcional):** si un consumible de vigencia y un escudo cubren el mismo verbo, hay que decidir cuál prevalece. Mitigación: la vigencia tiene prioridad y no consume cargas del escudo mientras esté activa — decisión a confirmar con el equipo antes de programar esta historia.

