# [G11 — Recibir un aviso antes de que venza un ítem]

> **Taiga Ref:** #786 | **ID:** 9552676
> **Épica:** [#770 — G11 — Vencimiento de Ítems del Inventario](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Recibir un aviso antes de que venza un ítem

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO
*   **Quiero**: recibir un aviso cuando un ítem mío está por vencer
*   **Para**: usarlo antes de perderlo

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: Mercado avisa 24 horas antes del vencimiento de cada ítem no usado. El aviso se envía una sola vez por ítem. Si el ítem se usa antes, no se avisa.
*   [ ] Validaciones: no se avisa por ítems ya usados, vencidos o congelados.
*   [ ] Datos obligatorios: alumno, curso, ítem, fecha de vencimiento.
*   [ ] Performance (tiempos, volumen, límites): el aviso sale con un margen máximo de 15 minutos respecto de las 24 horas.
*   [ ] Seguridad (roles, permisos, datos sensibles): cada alumno recibe solo avisos de sus ítems.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el aviso se puede leer con lector de pantalla.
*   [ ] Otros: si varios ítems vencen el mismo día, se agrupan en un solo aviso.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Se envía un aviso 24 horas antes del vencimiento de cada ítem no usado.
*   [ ] CA2: Cada ítem genera como máximo un aviso.
*   [ ] CA3: No se avisa por ítems usados, vencidos o congelados.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Aviso previo al vencimiento

**Escenario 1: aviso normal**

*   **Dado**: que tengo un escudo sin usar que vence mañana a las 18
*   **Cuando**: son las 18 de hoy
*   **Entonces**: recibo "Tu Escudo Reforzado vence en 24 horas"

**Escenario 2: ítem ya usado**

*   **Dado**: que usé el escudo antes de las 24 horas previas
*   **Cuando**: llega el momento del aviso
*   **Entonces**: no recibo ningún aviso

**Escenario 3: varios ítems**

*   **Dado**: que tengo tres ítems que vencen mañana
*   **Cuando**: llega el momento del aviso
*   **Entonces**: recibo un solo aviso con los tres ítems

* * *

## Prototipo

*   **Mock API / Swagger**: evento `ITEM_EXPIRING_SOON` (nombre a confirmar con Notificaciones)

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 2
*   **Prioridad (MoSCoW / Numérica)**: Could

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Notificaciones.
*   Módulos afectados: Inventario.
*   Otros equipos / aprobaciones: equipo de Notificaciones (formato del aviso).
*   Impacto en datos / migraciones: campo `expiry_notice_sent` en el inventario.
*   Riesgos y mitigación (opcional): avisos duplicados por reintentos → marca de aviso enviado por ítem.


