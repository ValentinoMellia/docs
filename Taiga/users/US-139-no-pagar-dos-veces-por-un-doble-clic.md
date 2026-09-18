# [G11 — No pagar dos veces por un doble clic]

> **Taiga Ref:** #139 | **ID:** 9539961
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** que un doble clic o un problema de red no me cobren dos veces
*   **Para:** confiar en que mis monedas no desaparecen por un error técnico

Notas / Observaciones
---------------------

*   **Reglas de negocio:** toda operación que mueve monedas acepta una clave de idempotencia generada por el cliente. Reintentar con la misma clave devuelve el resultado original, nunca cobra de nuevo.
*   **Validaciones:** la misma clave con un contenido distinto es un conflicto, no un reintento — se rechaza sin cobrar.
*   **Datos obligatorios:** clave de idempotencia (única), ID del ALUMNO, ID de la oferta del catálogo, hash del contenido de la petición.
*   **Performance (tiempos, volumen, límites):** la verificación de idempotencia debe resolverse con un índice único, no con una búsqueda secuencial.
*   **Seguridad (roles, permisos, datos sensibles):** la clave la genera el cliente pero se valida contra el ALUMNO autenticado — una clave no puede reutilizarse entre alumnos.
*   **Accesibilidad (WCAG/teclado/lectores):** el botón deshabilitado durante el proceso debe anunciar su estado a lectores de pantalla, no solo cambiar de color.
*   **Otros:** la protección del front (botón deshabilitado) no reemplaza la del back; dos pestañas son dos intentos legítimos y el front no los ve.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** una misma compra enviada dos veces con la misma clave de idempotencia crea una sola orden y realiza un solo cobro.
*   **CA2:** una clave ya usada pero con contenido distinto se rechaza como conflicto, sin cobrar nada.
*   **CA3:** mientras la compra está en curso, la interfaz deshabilita la acción de confirmar con un indicador de progreso.
*   **Extras (opcional):** existe una prueba automatizada que envía la misma petición en paralelo y verifica que solo se crea una orden.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Garantía de idempotencia en las operaciones de compra

**Escenario 1**

*   **Dado:** que el cliente generó una clave de idempotencia para la compra
*   **Cuando:** la petición llega dos veces con esa misma clave
*   **Entonces:** se crea una única orden, se realiza un único descuento y la segunda respuesta es idéntica a la primera

**Escenario 2**

*   **Dado:** que la clave ya se usó para comprar un escudo
*   **Cuando:** llega una petición con esa clave pero pidiendo otro ítem
*   **Entonces:** el sistema responde con un conflicto y no cobra ni entrega nada

**Escenario 3**

*   **Dado:** que el ALUMNO confirmó la compra
*   **Cuando:** la operación todavía está en curso
*   **Entonces:** el botón de confirmar queda deshabilitado con un indicador de progreso

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** mismo endpoint de la historia anterior, documentando el encabezado de idempotencia y el código de conflicto

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño de su propia idempotencia).
*   **Módulos afectados:** Órdenes.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** tabla de claves consumidas con índice único, más el hash del contenido para detectar el conflicto.
*   **Riesgos y mitigación (opcional):** el error más común es generar una clave nueva en cada reintento, que anula toda la protección. Mitigación: exigir un test que reintenta con la misma clave antes de cerrar la historia.

