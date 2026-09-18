# [G11 — Ver cuándo vencen mis ítems]

> **Taiga Ref:** #785 | **ID:** 9552675
> **Épica:** [#770 — G11 — Vencimiento de Ítems del Inventario](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Ver cuándo vencen mis ítems

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO
*   **Quiero**: ver en el catálogo y en mi inventario cuándo vence cada ítem
*   **Para**: decidir qué comprar y usar primero

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: en el catálogo se muestra la regla de vencimiento del ítem ("vence el 30/10" o "vence 7 días después de comprarlo"). En el inventario se muestra la fecha exacta de cada ítem. Los ítems sin vencimiento dicen "no vence". El inventario se puede ordenar por fecha de vencimiento.
*   [ ] Validaciones: la fecha se calcula con la hora del servidor.
*   [ ] Datos obligatorios: fecha de vencimiento de cada ítem del inventario.
*   [ ] Performance (tiempos, volumen, límites): el inventario carga en menos de 1 segundo.
*   [ ] Seguridad (roles, permisos, datos sensibles): cada alumno ve solo sus ítems.
*   [ ] Accesibilidad (WCAG/teclado/lectores): la fecha se muestra como texto, no solo con un color o ícono.
*   [ ] Otros: los ítems que vencen en menos de 24 horas se destacan.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: El catálogo muestra la regla de vencimiento de cada ítem que la tenga.
*   [ ] CA2: El inventario muestra la fecha exacta de vencimiento de cada ítem.
*   [ ] CA3: Los ítems que vencen en menos de 24 horas aparecen destacados.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Visualización del vencimiento

**Escenario 1: antes de comprar**

*   **Dado**: que el "Escudo Reforzado" vence 7 días después de comprarlo
*   **Cuando**: lo veo en el catálogo
*   **Entonces**: leo "vence 7 días después de comprarlo"

**Escenario 2: en el inventario**

*   **Dado**: que compré ese escudo el 16 de septiembre
*   **Cuando**: abro mi inventario
*   **Entonces**: veo que vence el 23 de septiembre

**Escenario 3: ítem por vencer**

*   **Dado**: que un ítem vence mañana a las 10
*   **Cuando**: abro mi inventario
*   **Entonces**: lo veo destacado con "vence en menos de 24 horas"

* * *

## Prototipo

*   **Mock API / Swagger**: `GET /api/v1/market/inventory?courseId={id}` (campo `expirationDatetime`)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 2
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado.
*   Módulos afectados: Catálogo, Inventario (front).
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: ninguno.
*   Riesgos y mitigación (opcional): —


