# [G11 — No pagar dos veces por un doble clic]

> **Taiga Ref:** #139 | **ID:** 9539961
> **Épica:** [#137 — G11 — Compra Directa con Doble Reserva](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** 5
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

*   **Como:** ALUMNO
*   **Quiero:** que un doble clic o un corte de red no me generen dos compras
*   **Para:** confiar en que mis monedas no desaparecen por un error técnico

### Notas / Observaciones

*   **Reglas de negocio:** la compra acepta una **clave de idempotencia** generada por el cliente. Reintentar con la misma clave devuelve el resultado original y nunca crea una segunda orden. **Hay que agregar esa clave al contrato del endpoint de compra, que hoy no la recibe.**
*   **Validaciones:** la misma clave con un contenido distinto es un conflicto, no un reintento: se rechaza sin comprar nada.
*   **Datos obligatorios:** clave de idempotencia, alumno, oferta y huella del contenido de la petición.
*   **Performance (tiempos, volumen, límites):** la verificación se resuelve con un índice único, no con una búsqueda secuencial.
*   **Seguridad (roles, permisos, datos sensibles):** la clave se valida contra el ALUMNO autenticado; una clave no puede reutilizarse entre alumnos.
*   **Accesibilidad (WCAG/teclado/lectores):** el botón deshabilitado durante el proceso anuncia su estado, no solo cambia de color.
*   **Otros:** la protección del front no reemplaza a la del backend: dos pestañas son dos intentos legítimos que el front no ve.

### Criterios de Aceptación (CA)

*   **CA1:** la misma compra enviada dos veces con la misma clave crea una sola orden y reserva las monedas una sola vez.
*   **CA2:** una clave ya usada con un contenido distinto se rechaza como conflicto, sin comprar nada.
*   **CA3:** mientras la compra está en curso, la interfaz deshabilita el botón y muestra que está procesando.
*   **Extras (opcional):** hay una prueba automatizada que envía la misma petición en paralelo y verifica que se cree una sola orden.

### BDD (mínimo 3 escenarios)

**Característica:** Idempotencia de la compra

**Escenario 1**

*   **Dado:** una compra enviada con una clave de idempotencia
*   **Cuando:** la misma petición llega dos veces con esa clave
*   **Entonces:** se crea una única orden y la segunda respuesta es idéntica a la primera

**Escenario 2**

*   **Dado:** una clave ya usada para comprar un escudo
*   **Cuando:** llega una petición con esa misma clave pero para otra oferta
*   **Entonces:** el sistema responde con un conflicto y no compra nada

**Escenario 3**

*   **Dado:** un ALUMNO que confirmó la compra
*   **Cuando:** la operación todavía está en curso
*   **Entonces:** el botón queda deshabilitado con un indicador de progreso

### Prototipo

*   **Mock API / Swagger:** `POST /api/v1/market/orders` con la cabecera de idempotencia y su código de conflicto

### Estimación / Prioridad

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Must / 1

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Must / 1 |

### Dependencias / Impactos

*   **Servicios involucrados:** Mercado (dueño de su propia idempotencia).
*   **Módulos afectados:** Mercado — Órdenes.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** tabla de claves consumidas con índice único y huella del contenido.
*   **Riesgos y mitigación:** generar una clave nueva en cada reintento anula toda la protección; se mitiga exigiendo una prueba que reintente con la misma clave.

> **Pendiente detectado en la propia historia:** el endpoint `POST /api/v1/market/orders` todavía no recibe la clave de idempotencia en su contrato actual — hay que agregarla antes de poder implementar esta historia. Ver [`CONTRATOS-COMUNICACION-SPRINT1.md`](../../CONTRATOS-COMUNICACION-SPRINT1.md) §1.2.
