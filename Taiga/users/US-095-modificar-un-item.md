# [G11 — Editar una oferta publicada]

> **Taiga Ref:** #95 | **ID:** 9535768
> **Épica:** [#90 — G11 — Catálogo Abierto por Plantillas](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** 3
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

*   **Como:** PROFESOR
*   **Quiero:** modificar una oferta que ya publiqué en mi curso
*   **Para:** corregir su precio, su descripción o su configuración sin tener que darla de baja

### Notas / Observaciones

*   **Reglas de negocio:** se pueden editar nombre, descripción, precio, stock y los parámetros del tipo de plantilla. **No** se puede cambiar la plantilla base de una oferta ya publicada: para eso se publica una nueva.
*   **Validaciones:** las mismas que al publicar (precio ≥ 1, stock vacío o mayor a 0, configuración completa según el tipo). La oferta debe existir y ser del curso del PROFESOR.
*   **Datos obligatorios:** identificador de la oferta y los campos a modificar.
*   **Performance (tiempos, volumen, límites):** actualización transaccional.
*   **Seguridad (roles, permisos, datos sensibles):** solo el PROFESOR del curso; cualquier otro intento se rechaza.
*   **Accesibilidad (WCAG/teclado/lectores):** formulario navegable por teclado, con los errores asociados a su campo.
*   **Otros:** un cambio de precio rige solo hacia adelante (RF-CFG-06): las compras ya hechas conservan el precio con el que se ejecutaron.

### Criterios de Aceptación (CA)

*   **CA1:** el PROFESOR modifica una oferta de su curso y los cambios se reflejan en la vitrina.
*   **CA2:** una edición inválida (precio en cero, configuración incompleta) se rechaza indicando el campo.
*   **CA3:** cambiar el precio no altera las órdenes ya realizadas con el precio anterior.
*   **Extras (opcional):** cada edición queda registrada con autor y fecha.

### BDD (mínimo 3 escenarios)

**Característica:** Edición de ofertas del catálogo

**Escenario 1**

*   **Dado:** una oferta activa en el curso del PROFESOR
*   **Cuando:** cambia su precio y confirma
*   **Entonces:** la vitrina muestra el precio nuevo

**Escenario 2**

*   **Dado:** una oferta de tipo escudo
*   **Cuando:** el PROFESOR intenta dejarla sin cargas
*   **Entonces:** el sistema rechaza la edición e indica el dato faltante

**Escenario 3**

*   **Dado:** una compra ya realizada con el precio anterior
*   **Cuando:** el PROFESOR cambia el precio de esa oferta
*   **Entonces:** la orden anterior conserva el precio con el que se ejecutó

### Prototipo

*   **Mock API / Swagger:** `PUT /api/v1/market/courses/{courseId}/catalog/items/{itemId}`

### Estimación / Prioridad

*   **Puntos (Fibonacci):** 3
*   **Prioridad (MoSCoW / Numérica):** Must / 1

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 3 | Must / 1 |

### Dependencias / Impactos

*   **Servicios involucrados:** Mercado (dueño). Usuarios y Cursos — simulados por ahora.
*   **Módulos afectados:** Mercado — Catálogo.
*   **Otros equipos / aprobaciones:** ninguna bloqueante.
*   **Impacto en datos / migraciones:** actualización sobre la tabla de ofertas.
*   **Riesgos y mitigación:** que una edición afecte compras pasadas; se mitiga con el precio guardado en cada orden.
