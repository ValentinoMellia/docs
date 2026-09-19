# [G11 — Comprar una oferta del catálogo]

> **Taiga Ref:** #138 | **ID:** 9539955
> **Épica:** [#137 — G11 — Compra Directa con Doble Reserva](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** 8
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

*   **Como:** ALUMNO
*   **Quiero:** canjear mis monedas por una oferta del catálogo de mi curso
*   **Para:** obtener el ítem sin riesgo de perder el saldo si la operación falla a mitad de camino

### Notas / Observaciones

*   **Reglas de negocio:** la compra es asíncrona y pasa por 4 fases — reservar el stock (si la oferta es limitada), reservar las monedas en Banco, acreditar el ítem en el inventario de Banco y recién ahí confirmar el débito. **Nunca se cobra antes de que el ítem esté acreditado.**
*   **Validaciones:** antes de reservar nada se verifica que la oferta esté activa y sea del curso del ALUMNO, y que las monedas sean de ese mismo curso (RF-INT-04).
*   **Datos obligatorios:** curso-cohorte, alumno, oferta, precio aplicado, estado de la orden e identificadores de las reservas de monedas y de stock.
*   **Performance (tiempos, volumen, límites):** la respuesta inicial es inmediata; el resto de la saga se resuelve en segundo plano.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO compra únicamente con su propia cuenta; toda operación que mueve monedas queda auditada.
*   **Accesibilidad (WCAG/teclado/lectores):** la confirmación de compra es operable por teclado y anuncia el cambio de estado a lectores de pantalla.
*   **Otros:** los criterios describen el resultado observable, así siguen siendo válidos cuando el Banco real reemplace al simulador.

### Criterios de Aceptación (CA)

*   **CA1:** con saldo suficiente, el ALUMNO confirma la compra y termina con el ítem acreditado en su inventario y las monedas descontadas una sola vez.
*   **CA2:** si no le alcanza el saldo, la compra se rechaza, no se cobra nada y se libera el stock reservado.
*   **CA3:** si la oferta está inactiva o es de otro curso, la compra se rechaza antes de reservar monedas o stock.
*   **Extras (opcional):** la orden guarda el precio con el que se ejecutó, de modo que un cambio posterior no altera el historial.

### BDD (mínimo 3 escenarios)

**Característica:** Compra de una oferta con reserva de monedas, acreditación del ítem y cobro final

**Escenario 1**

*   **Dado:** un ALUMNO con saldo suficiente y una oferta activa de su curso
*   **Cuando:** confirma la compra
*   **Entonces:** se le reservan las monedas, se le acredita el ítem y recién después se le cobra, quedando la orden confirmada

**Escenario 2**

*   **Dado:** un ALUMNO sin saldo suficiente
*   **Cuando:** confirma la compra
*   **Entonces:** la orden queda rechazada, no se le cobra nada y el stock reservado vuelve a estar disponible

**Escenario 3**

*   **Dado:** una oferta que el profesor desactivó
*   **Cuando:** el ALUMNO intenta comprarla
*   **Entonces:** la compra se rechaza sin haber reservado monedas ni stock

### Prototipo

*   **Mock API / Swagger:** `POST /api/v1/market/orders`

### Estimación / Prioridad

*   **Puntos (Fibonacci):** 8
*   **Prioridad (MoSCoW / Numérica):** Must / 1

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 8 | Must / 1 |

### Dependencias / Impactos

*   **Servicios involucrados:** Mercado (orquesta). Banco (reserva, acreditación y débito) — simulado por ahora. Usuarios y Cursos — simulados.
*   **Módulos afectados:** Mercado — Catálogo y Órdenes.
*   **Otros equipos / aprobaciones:** Banco, para cerrar el contrato de comandos y eventos.
*   **Impacto en datos / migraciones:** tabla de órdenes con su estado, precio aplicado e identificadores de reservas.
*   **Riesgos y mitigación:** que la compra quede a mitad de camino si Banco no responde; se mitiga con la liberación de reservas y la reconciliación de órdenes colgadas.

> Reemplaza al enfoque anterior, que descontaba de un saldo simulado dentro de Mercado. Ahora Mercado orquesta y Banco es dueño de las monedas y del inventario. Ver el detalle completo de la saga en [`CONTRATOS-COMUNICACION-SPRINT1.md`](../../CONTRATOS-COMUNICACION-SPRINT1.md) §3.1.
