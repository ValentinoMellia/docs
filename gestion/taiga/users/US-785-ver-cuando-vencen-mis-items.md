# [G11 — Saber cuánto dura un ítem antes de comprarlo]

> **Taiga Ref:** #785 | **ID:** 9552675
> **Épica:** [#770 — G11 — Vencimiento de Ítems](../epics/EPIC-770-vencimiento-de-items-del-inventario.md)
> **Estado:** New | **Puntos:** 2
> **Asignado a:** Sin asignar | **Propietario:** Mateo Nicolas Presset

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

**Como** alumno de un curso,
**quiero** ver cuánto me va a durar un ítem antes de gastarme las monedas,
**para** no comprar algo que se me va a vencer antes de que lo necesite.

### Notas / Observaciones

*   **Reglas de negocio:** la vitrina del curso y el detalle de cada oferta muestran cuánto dura el ítem. Si la oferta no tiene vencimiento, se dice que el ítem no vence. Si lo tiene, se dice en cuánto tiempo vence contado desde la compra, o hasta qué día sirve.
*   **Validaciones:** el dato que se muestra es siempre el de la oferta tal como está publicada en ese momento. Si el profesor la cambia, la vitrina muestra el valor nuevo de ahí en más.
*   **Datos obligatorios:** oferta, curso y el vencimiento configurado, o la indicación de que no tiene.
*   **Performance:** el dato viaja junto con el resto de la oferta, así que no agrega ninguna consulta extra ni demora la vitrina.
*   **Seguridad:** el alumno solo ve las ofertas del curso que está cursando, igual que con el resto de la vitrina.
*   **Accesibilidad:** el vencimiento se muestra como texto entendible, del estilo "vence a los 15 días de comprarlo", no como una fecha suelta sin contexto ni como un ícono de reloj sin explicación. La información no se transmite solo con color.
*   **Otros:** esta historia es sobre lo que el alumno ve antes de comprar, que es del Mercado. Ver los ítems que ya tiene y cuándo le vencen es una pantalla del inventario, y el inventario es de Banco.

### Criterios de Aceptación (CA)

*   **CA1:** una oferta con vencimiento muestra, tanto en la vitrina como en el detalle, cuánto dura el ítem.
*   **CA2:** una oferta sin vencimiento indica de forma explícita que el ítem no vence.
*   **CA3:** si el profesor cambia el vencimiento de la oferta, la vitrina pasa a mostrar el valor nuevo.
*   **Extras (opcional):** poder filtrar la vitrina para ver solo las ofertas sin vencimiento.

### BDD (mínimo 3 escenarios)

**Escenario 1: oferta con vencimiento**
*   **Dado** que el profesor publicó un escudo que vence a los 15 días
*   **Cuando** el alumno abre la vitrina de su curso
*   **Entonces** ve que ese escudo vence a los 15 días de comprarlo

**Escenario 2: oferta sin vencimiento**
*   **Dado** que el profesor publicó un boost sin vencimiento
*   **Cuando** el alumno abre el detalle de esa oferta
*   **Entonces** ve indicado que ese ítem no vence

**Escenario 3: el profesor cambia el plazo**
*   **Dado** que una oferta pasó de 15 a 5 días de vencimiento
*   **Cuando** el alumno vuelve a abrir la vitrina
*   **Entonces** ve el plazo de 5 días

### Prototipo

*   **Mock API / Swagger:** `GET /api/cursos/{cursoId}/vitrina` y `GET /api/ofertas/{ofertaId}`, con el vencimiento entre los datos de la oferta.

### Estimación / Prioridad

| Estimación (Fibonacci) | Prioridad (MoSCoW) |
|---|---|
| 2 | Should |

### Dependencias / Impactos

*   **Catálogo:** depende de que la oferta tenga el vencimiento configurado y de que la vitrina y el detalle lo muestren.
*   **Banco:** dueño del inventario. Ver los ítems ya comprados y su vencimiento es una pantalla de ese módulo, no de este.
*   **Cursos:** dice a qué curso pertenece el alumno, que es lo que define qué vitrina ve.

> Esta historia ya estaba correctamente acotada al alcance de Mercado (Sección 12-C de `CONTEXTO-MERCADO-SPRINT1.md`) al momento de sincronizar este archivo — no hizo falta recortarla, el PO ya la había reescrito con el límite correcto.
