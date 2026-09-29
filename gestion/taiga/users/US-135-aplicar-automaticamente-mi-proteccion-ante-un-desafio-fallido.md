# [G11 — Aplicar automáticamente mi protección ante un desafío fallido]

> **Taiga Ref:** #135 | **ID:** 9539946
> **Épica:** [#131 — G11 — Inventario, Equipamiento y Consumo del Alumno](../epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** que mi ítem de protección equipado se consuma automáticamente cuando fallo un desafío
*   **Para:** que el sistema neutralice el castigo sin que yo tenga que hacer nada manualmente

Notas / Observaciones
---------------------

*   **Reglas de negocio:** Mercado devuelve **hechos sobre su propio dominio** (qué verbo de efecto corresponde), nunca una orden sobre el dominio ajeno (vidas, XP). El consumo es de uso único (RF-REC-05) y el alcance es siempre por curso (RF-REC-01).
*   **Validaciones:** el ALUMNO debe pertenecer al curso/cohorte consultado. El "momento" recibido debe pertenecer al vocabulario provisorio definido por el equipo (ver Suposiciones de la épica).
*   **Datos obligatorios:** ID del ALUMNO, curso/cohorte, momento del efecto, identificador de contexto (para idempotencia).
*   **Performance (tiempos, volumen, límites):** objetivo interno p95 < 200 ms, ya que en el futuro esta consulta ocurrirá con el ALUMNO esperando el resultado de una entrega.
*   **Seguridad (roles, permisos, datos sensibles):** operación pensada para ser invocada entre servicios (por el futuro Motor de Desafíos); hoy se prueba simulando esa invocación, ya que ese servicio todavía no existe.
*   **Accesibilidad (WCAG/teclado/lectores):** no aplica — no tiene interfaz propia.
*   **Otros:** ante una consulta repetida con el mismo identificador de contexto (reintento), debe devolver la misma respuesta sin consumir un segundo ítem.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** si el ALUMNO tiene una protección equipada para el momento consultado, el sistema devuelve el verbo correspondiente y consume el ítem en la misma operación.
*   **CA2:** si el ALUMNO no tiene ningún efecto activo, el sistema responde exitosamente con una lista vacía, nunca con un error.
*   **CA3:** ante una consulta repetida con el mismo identificador de contexto, el sistema devuelve la misma respuesta sin consumir un segundo ítem.
*   **Extras (opcional):** la respuesta incluye un texto breve para mostrarle al ALUMNO qué pasó.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Resolución de efectos activos del inventario ante un fallo, con consumo atómico e idempotente

**Escenario 1**

*   **Dado:** que el ALUMNO tiene un escudo equipado en su curso/cohorte
*   **Cuando:** se consulta si tiene efectos activos para un fallo
*   **Entonces:** el sistema devuelve el verbo que evita la pérdida de vida y marca el ítem como consumido en la misma operación

**Escenario 2**

*   **Dado:** que el ALUMNO no tiene ningún ítem equipado ni ningún efecto vigente
*   **Cuando:** se consulta si tiene efectos activos
*   **Entonces:** el sistema devuelve una respuesta exitosa con la lista vacía, sin error

**Escenario 3**

*   **Dado:** que ya se resolvió una consulta para un contexto y se consumió un escudo
*   **Cuando:** llega otra vez la misma consulta con el mismo identificador de contexto
*   **Entonces:** el sistema devuelve exactamente la misma respuesta sin consumir un segundo ítem

Prototipo
---------

*   **Capturas:** no aplica — la historia no tiene pantalla propia.
*   **URL Figma:** no aplica.
*   **Libro de cuentos:** no aplica.
*   **API simulada / Swagger:** `POST /api/market/inventory/resolve-effects`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 13
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 13 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño y proveedor). Motor de Desafíos y Roadmap serán consumidores futuros — hoy se prueba con una invocación simulada.
*   **Módulos afectados:** Mercado (Inventario).
*   **Otros equipos / aprobaciones:** ninguna bloqueante hoy; acordar el vocabulario definitivo de verbos apenas Motor de Desafíos y Roadmap tengan con quién negociarlo.
*   **Impacto en datos / migraciones:** agrega marca de consumo al ítem de inventario y una tabla de contextos ya procesados para la idempotencia.
*   **Riesgos y mitigación (opcional):** dos fallos casi simultáneos podrían consumir la misma instancia. Mitigación: verificación y consumo en la misma transacción, con prueba de concurrencia obligatoria.

* * *

