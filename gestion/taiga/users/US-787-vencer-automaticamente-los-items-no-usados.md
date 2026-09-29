# [G11 — Vencer automáticamente los ítems no usados]

> **Taiga Ref:** #787 | **ID:** 9552679
> **Épica:** [#770 — G11 — Vencimiento de Ítems del Inventario](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Vencer automáticamente los ítems no usados

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: que los ítems no usados venzan solos al llegar su fecha
*   **Para**: que la regla de vencimiento se cumpla sin intervención manual

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: un proceso automático pasa a "vencido" los ítems disponibles o equipados cuya fecha de vencimiento ya pasó. Los desequipa, publica un aviso de ítem vencido y no devuelve monedas. Aunque el proceso se atrase, la consulta de Desafíos ya no devuelve ítems vencidos.
*   [ ] Validaciones: cada ítem se vence una sola vez, aunque haya más de una instancia de Mercado.
*   [ ] Datos obligatorios: fecha real de vencimiento.
*   [ ] Performance (tiempos, volumen, límites): ningún ítem sigue disponible más de 1 minuto después de su vencimiento.
*   [ ] Seguridad (roles, permisos, datos sensibles): proceso interno, sin acceso de usuarios.
*   [ ] Accesibilidad (WCAG/teclado/lectores): no aplica.
*   [ ] Otros: el ítem vencido queda en el historial del alumno (baja lógica).

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Ningún ítem sigue disponible más de 1 minuto después de vencer.
*   [ ] CA2: Un ítem vencido no se informa a Desafíos, aunque el proceso se haya atrasado.
*   [ ] CA3: El aviso de ítem vencido se publica una sola vez por ítem.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Vencimiento automático

**Escenario 1: vencimiento**

*   **Dado**: que un escudo sin usar vence a las 18:00
*   **Cuando**: corre el proceso a las 18:00:30
*   **Entonces**: el escudo pasa a vencido y se publica el aviso

**Escenario 2: ítem equipado**

*   **Dado**: que tengo equipado un escudo que vence hoy
*   **Cuando**: llega la hora de vencimiento
*   **Entonces**: el escudo se desequipa y queda vencido

**Escenario 3: dos instancias**

*   **Dado**: que dos instancias de Mercado corren el proceso a la vez
*   **Cuando**: encuentran el mismo ítem vencido
*   **Entonces**: se vence una sola vez

* * *

## Prototipo

*   **Mock API / Swagger**: evento `ITEM_EXPIRED` (tópico `mercado.inventario`)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Motor de Desafíos G03, Notificaciones.
*   Módulos afectados: Inventario.
*   Otros equipos / aprobaciones: G03 (no usar ítems vencidos).
*   Impacto en datos / migraciones: estado "vencido" en el inventario.
*   Riesgos y mitigación (opcional): ítems usados justo al vencer → se toma como válida la hora del servidor al procesar el uso.


