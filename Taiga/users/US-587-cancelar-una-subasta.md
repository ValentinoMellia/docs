# [G11 — Cancelar una subasta]

> **Taiga Ref:** #587 | **ID:** 9549027
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Cancelar una subasta

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: PROFESOR
*   **Quiero**: cancelar una subasta que todavía no terminó
*   **Para**: corregir un error sin perjudicar a nadie

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: al cancelar, nadie recibe el ítem y se devuelven todas las monedas retenidas (RF-INT-06). Se avisa a todos los que ofertaron.
*   [ ] Validaciones: solo se pueden cancelar subastas programadas o abiertas, no las que se están cerrando o ya cerraron. El motivo es obligatorio.
*   [ ] Datos obligatorios: motivo de la cancelación.
*   [ ] Performance (tiempos, volumen, límites): devoluciones pedidas en menos de 1 minuto.
*   [ ] Seguridad (roles, permisos, datos sensibles): solo un PROFESOR del curso. Se registra quién canceló y cuándo.
*   [ ] Accesibilidad (WCAG/teclado/lectores): se pide confirmación antes de cancelar, con foco en el botón "Volver".
*   [ ] Otros: la subasta cancelada sigue visible en el historial.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Al cancelar, todas las monedas retenidas se devuelven.
*   [ ] CA2: No se puede cancelar una subasta que se está cerrando o que ya cerró.
*   [ ] CA3: Queda registrado quién canceló, cuándo y por qué.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Cancelación de subasta por el profesor

**Escenario 1: cancelación con ofertas**

*   **Dado**: que una subasta abierta tiene 3 ofertas
*   **Cuando**: la cancelo con el motivo "Ítem cargado por error"
*   **Entonces**: nadie recibe el ítem, se devuelven las monedas de los 3 alumnos y reciben un aviso

**Escenario 2: cancelación de una subasta programada**

*   **Dado**: que una subasta todavía no empezó
*   **Cuando**: la cancelo
*   **Entonces**: queda cancelada y no llega a abrirse

**Escenario 3: intento de cancelar durante el cierre**

*   **Dado**: que la subasta se está cerrando
*   **Cuando**: intento cancelarla
*   **Entonces**: el sistema me informa que ya no se puede cancelar

* * *

## Prototipo

*   **Mock API / Swagger**: `POST /api/v1/market/auctions/{id}/cancel`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 3
*   **Prioridad (MoSCoW / Numérica)**: Must

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Banco G08, Notificaciones.
*   Módulos afectados: Subastas.
*   Otros equipos / aprobaciones: —
*   Impacto en datos / migraciones: `auction` guarda motivo, usuario y fecha de cancelación.
*   Riesgos y mitigación (opcional): cancelar justo cuando empieza el cierre → el cambio de estado se controla para que solo pase una de las dos cosas.


