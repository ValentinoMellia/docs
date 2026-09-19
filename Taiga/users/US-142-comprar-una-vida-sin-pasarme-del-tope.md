# [G11 — Comprar una vida sin pasarme del tope]

> **Taiga Ref:** #142 | **ID:** 9539965
> **Épica:** [#137 — G11 — Compra Directa con Doble Reserva](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** 5
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

*   **Como:** ALUMNO
*   **Quiero:** que no me dejen comprar una vida si ya llegué al máximo permitido
*   **Para:** no gastar monedas en algo que no voy a poder recibir

### Notas / Observaciones

*   **Reglas de negocio:** las vidas viven en el inventario de Banco, así que el tope lo valida Banco al momento de acreditar. Mercado reacciona al rechazo liberando las reservas, sin cobrar nada.
*   **Validaciones:** el motivo del rechazo tiene que llegar diferenciado, para poder explicarle al alumno que fue por el tope y no por saldo.
*   **Datos obligatorios:** alumno, curso-cohorte, oferta de tipo vida y motivo del rechazo.
*   **Performance (tiempos, volumen, límites):** sin exigencias particulares; es un paso más de la saga de compra.
*   **Seguridad (roles, permisos, datos sensibles):** la verificación es siempre sobre el propio ALUMNO.
*   **Accesibilidad (WCAG/teclado/lectores):** el mensaje de rechazo por tope se distingue claramente del de saldo insuficiente.
*   **Otros:** **a acordar con Banco** el motivo de rechazo específico cuando el alumno está en el tope.

### Criterios de Aceptación (CA)

*   **CA1:** si el ALUMNO está por debajo del tope, la compra de una vida se completa normalmente.
*   **CA2:** si ya está en el tope, la compra se rechaza, se liberan las reservas y no se le cobra nada.
*   **CA3:** el mensaje por tope se distingue del mensaje por saldo insuficiente.
*   **Extras (opcional):** la vitrina muestra la vida atenuada cuando el alumno ya está en el tope.

### BDD (mínimo 3 escenarios)

**Característica:** Compra de vidas respetando el tope vigente por curso

**Escenario 1**

*   **Dado:** un ALUMNO con menos vidas que el máximo permitido
*   **Cuando:** compra una vida
*   **Entonces:** se le acredita y se le cobra normalmente

**Escenario 2**

*   **Dado:** un ALUMNO que ya está en el máximo de vidas
*   **Cuando:** intenta comprar otra
*   **Entonces:** la compra se rechaza por el tope, se liberan las reservas y no se le cobra nada

**Escenario 3**

*   **Dado:** los dos motivos posibles de rechazo
*   **Cuando:** se produce cada uno por separado
*   **Entonces:** el ALUMNO recibe mensajes distintos para el tope y para el saldo insuficiente

### Prototipo

*   **Mock API / Swagger:** reutiliza `POST /api/v1/market/orders`, con el motivo de rechazo por tope en la respuesta de la saga

### Estimación / Prioridad

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Should / 2

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Should / 2 |

### Dependencias / Impactos

*   **Servicios involucrados:** Mercado (orquesta). Banco (dueño de las vidas y del tope) — simulado por ahora.
*   **Módulos afectados:** Mercado — Órdenes.
*   **Otros equipos / aprobaciones:** Banco, para acordar el motivo de rechazo por tope.
*   **Impacto en datos / migraciones:** ninguno adicional.
*   **Riesgos y mitigación:** que el rechazo llegue sin motivo diferenciado y el alumno no entienda por qué no pudo comprar; se mitiga acordando el motivo en el contrato.

> **Cambio de fondo respecto a la versión anterior:** el tope de vidas ya no es un valor simulado dentro de Mercado — el tope lo valida Banco al acreditar (consistente con CONTEXTO §9.7). Ver [`CONTRATOS-COMUNICACION-SPRINT1.md`](../../CONTRATOS-COMUNICACION-SPRINT1.md) §4, task #1002/#1003.
