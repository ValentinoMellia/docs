# [G11 — Comprar un ítem del catálogo]

> **Taiga Ref:** #138 | **ID:** 9539955
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** canjear mis monedas por un ítem publicado en el catálogo de mi curso
*   **Para:** obtenerlo sin arriesgarme a perder el saldo si la operación falla a mitad de camino

Notas / Observaciones
---------------------

*   **Reglas de negocio:** las monedas solo se canjean por vidas o equipamiento (RF-INT-01). Las monedas deben ser del mismo curso (RF-INT-04). La orden guarda el precio aplicado (RF-CFG-06).
*   **Validaciones:** en orden — ítem publicado y activo → ALUMNO pertenece al curso/cohorte → saldo simulado suficiente. Todo esto se valida **antes** de tocar el saldo.
*   **Datos obligatorios:** ID del curso/cohorte, ID del ALUMNO, ID del ítem, precio aplicado, clave de idempotencia, estado de la orden.
*   **Performance (tiempos, volumen, límites):** la respuesta al ALUMNO es inmediata; la confirmación puede resolverse en un paso posterior.
*   **Seguridad (roles, permisos, datos sensibles):** el ALUMNO solo compra con su propio saldo simulado. Toda operación que mueve monedas queda en log de auditoría.
*   **Accesibilidad (WCAG/teclado/lectores):** el modal de confirmación de compra debe ser operable por teclado, con el cambio de estado anunciado a lectores de pantalla.
*   **Otros:** los criterios describen el resultado observable, para que sigan siendo válidos el día que el saldo simulado se reemplace por el Banco real.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO con saldo suficiente puede confirmar la compra, se le descuenta el precio y recibe el ítem en su inventario.
*   **CA2:** si el ALUMNO no cumple una condición previa (ítem no publicado, o monedas de otro curso), la operación se rechaza sin mover ninguna moneda.
*   **CA3:** si el saldo del ALUMNO no alcanza, la operación se rechaza y el saldo queda intacto.
*   **Extras (opcional):** la orden registra el precio aplicado, de modo que un cambio posterior del parámetro no altera el historial.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Compra directa de un ítem del catálogo, con descuento de saldo y entrega al inventario

**Escenario 1**

*   **Dado:** que el ALUMNO tiene saldo simulado suficiente y el ítem está publicado y activo
*   **Cuando:** confirma la compra
*   **Entonces:** se le descuenta el precio del ítem, el ítem queda disponible en su inventario y la orden registra el precio con el que se ejecutó

**Escenario 2**

*   **Dado:** que el ALUMNO intenta comprar con monedas de otro curso
*   **Cuando:** confirma la compra
*   **Entonces:** la operación se rechaza sin haber movido ninguna moneda

**Escenario 3**

*   **Dado:** que el precio del ítem supera el saldo simulado del ALUMNO
*   **Cuando:** confirma la compra
*   **Entonces:** la orden queda rechazada, el saldo permanece sin cambios y no se crea ninguna instancia en su inventario

Prototipo
---------

*   **Capturas:** front funcionando en Angular con el botón "Comprar" ya presente en cada tarjeta del catálogo.
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `POST /api/market/orders`

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

*   **Servicios involucrados:** Mercado (orquesta, incluido el saldo simulado). Identidad y Cursos/Matrícula — mockeados.
*   **Módulos afectados:** Catálogo, Órdenes, Inventario.
*   **Otros equipos / aprobaciones:** ninguna bloqueante hoy.
*   **Impacto en datos / migraciones:** crea la tabla de órdenes y la de saldo simulado por alumno/curso.
*   **Riesgos y mitigación (opcional):** el riesgo es tener que rehacer esta historia cuando llegue el Banco real. Mitigación: implementar el saldo detrás de una interfaz propia desde el día uno.

