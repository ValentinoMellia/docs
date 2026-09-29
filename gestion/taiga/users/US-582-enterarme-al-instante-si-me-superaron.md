# [G11 — Enterarme al instante si me superaron]

> **Taiga Ref:** #582 | **ID:** 9549022
> **Épica:** [#577 — G11 — Subastas de Ítems con Tiempo Límite](../epics/EPIC-577-subastas-de-items-con-tiempo-limite.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

## G11 — Enterarme al instante si me superaron

* * *

## Descripción (Como / Quiero / Para)

*   **Como**: ALUMNO que participa en una subasta
*   **Quiero**: ver en el momento si alguien me superó y cuánto falta para que termine
*   **Para**: poder reaccionar a tiempo, aunque esté en el celular

* * *

## Notas / Observaciones

*   [ ] Reglas de negocio: la pantalla de la subasta se actualiza sola cuando hay una nueva oferta, cambia quién va ganando, la subasta termina o se cancela. Si me superan, además recibo una notificación en la plataforma.
*   [ ] Validaciones: solo se envían novedades de subastas del curso del alumno.
*   [ ] Datos obligatorios: oferta más alta, estado del alumno, tiempo restante.
*   [ ] Performance (tiempos, volumen, límites): los cambios llegan en menos de 2 segundos; soporta 120 usuarios conectados a la vez (RF-NFR-03).
*   [ ] Seguridad (roles, permisos, datos sensibles): no se muestra quién hizo cada oferta.
*   [ ] Accesibilidad (WCAG/teclado/lectores): el aviso "Te superaron" se anuncia al lector de pantalla.
*   [ ] Otros: si se corta la conexión, al volver la pantalla recupera lo que pasó mientras tanto.

* * *

## Criterios de Aceptación (CA)

*   [ ] CA1: Un cambio de quién va ganando se ve en menos de 2 segundos sin recargar la página.
*   [ ] CA2: Al recuperar la conexión, se muestran las novedades perdidas.
*   [ ] CA3: El alumno superado recibe también una notificación en la plataforma.

* * *

## BDD (mínimo 3 escenarios)

**Característica:** Seguimiento en vivo de una subasta

**Escenario 1: me superan**

*   **Dado**: que voy ganando una subasta y tengo la pantalla abierta
*   **Cuando**: otro alumno oferta más
*   **Entonces**: veo "Te superaron" con la nueva oferta más alta y recibo una notificación

**Escenario 2: se corta la conexión**

*   **Dado**: que estoy en el celular y pierdo señal por 1 minuto
*   **Cuando**: recupero la señal
*   **Entonces**: la pantalla se actualiza con lo que pasó en ese minuto

**Escenario 3: la subasta termina**

*   **Dado**: que estoy mirando la subasta
*   **Cuando**: llega la hora de cierre
*   **Entonces**: veo el resultado final sin recargar

* * *

## Prototipo

*   **Mock API / Swagger**: `GET /api/v1/market/auctions/{id}/stream` (actualización en vivo) — evento `OFERTA_SUPERADA`

* * *

## Estimación / Prioridad

*   **Puntos (Fibonacci)**: 5
*   **Prioridad (MoSCoW / Numérica)**: Should

* * *

## Dependencias / Impactos

*   Servicios involucrados: Mercado, Notificaciones.
*   Módulos afectados: Subastas (front web y móvil).
*   Otros equipos / aprobaciones: equipo de Notificaciones (formato del aviso "Te superaron").
*   Impacto en datos / migraciones: ninguno.
*   Riesgos y mitigación (opcional): muchas conexiones abiertas a la vez → solo quienes miran una subasta reciben sus novedades en vivo.


