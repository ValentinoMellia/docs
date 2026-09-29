# [G11 — Recuperar mis monedas si la compra no se completó]

> **Taiga Ref:** #141 | **ID:** 9539964
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** que mis monedas reservadas queden libres de nuevo si la compra no se pudo completar
*   **Para:** confiar en el Mercado aunque el sistema falle, sin perder saldo por algo que nunca terminó de cobrarse

Notas / Observaciones
---------------------

*   **Reglas de negocio (premisa corregida — decisión #8):** el débito nunca se confirma antes de corroborar que Grupo 12 puede acreditar el ítem (orden: reservar → `ITEM_PROVISION_REQUESTED`/`ITEM_PROVISIONED` → recién ahí `HOLD_CONFIRM_REQUESTED`). Por eso "se cobró pero no se acreditó el ítem" **no es un escenario del camino normal**: si Grupo 12 no puede acreditar el ítem, Mercado pide `HOLD_RELEASE_REQUESTED` y el hold se libera sin que se haya confirmado ningún débito. No hay monedas que reembolsar porque nunca se descontaron — es una liberación de reserva, no una devolución.
*   **Validaciones:** solo se libera el hold de una orden cuya acreditación de ítem falló (`ITEM_PROVISION_FAILED`) o cuyo hold expiró por TTL sin confirmarse.
*   **Datos obligatorios:** ID de la orden, ID del hold, estado, motivo de la liberación.
*   **Performance (tiempos, volumen, límites):** los reintentos de acreditación del ítem (antes de decidir liberar el hold) usan un esquema de espera creciente, para no saturar a Grupo 12.
*   **Seguridad (roles, permisos, datos sensibles):** la liberación del hold queda auditada con el motivo.
*   **Accesibilidad (WCAG/teclado/lectores):** el aviso al ALUMNO debe dejar claro en texto que la compra no se completó y que no se le cobró nada.
*   **Otros:** si `HOLD_RELEASE_REQUESTED` también falla tras agotar los reintentos, la orden queda visible en un listado para intervención manual — nunca se descarta en silencio.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** si Grupo 12 no puede acreditar el ítem, al agotarse los reintentos de acreditación Mercado libera el hold del ALUMNO (sin haber confirmado nunca un débito) y deja la orden en estado terminal `CANCELADA`.
*   **CA2:** una liberación de hold que se reintenta no descuenta ni acredita monedas — el saldo del ALUMNO nunca se movió mientras la orden no confirmó el débito.
*   **CA3:** si la liberación del hold tampoco se pudo completar, la orden queda marcada como pendiente de intervención y visible en un listado.
*   **Extras (opcional):** el ALUMNO recibe un aviso indicando que la compra no se completó y que no tuvo costo.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Liberación de holds de órdenes que no pudieron completarse

**Escenario 1**

*   **Dado:** que Grupo 12 no puede acreditar el ítem de mi orden (`ITEM_PROVISION_FAILED`)
*   **Cuando:** se agotan los reintentos de acreditación
*   **Entonces:** Mercado libera mi hold (`HOLD_RELEASE_REQUESTED` → `HOLD_RELEASED`) sin haberme cobrado nada, y la orden queda `CANCELADA`

**Escenario 2**

*   **Dado:** que ya se liberó el hold de una orden
*   **Cuando:** el proceso de liberación se ejecuta de nuevo por un reintento
*   **Entonces:** no se descuenta ni se acredita saldo por segunda vez

**Escenario 3**

*   **Dado:** que la liberación del hold no se pudo completar tras agotar los reintentos
*   **Cuando:** el proceso termina
*   **Entonces:** la orden queda marcada como pendiente de intervención manual en un listado de revisión

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** eventos `HOLD_RELEASE_REQUESTED` / `HOLD_RELEASED` contra Banco G08 (sin endpoint público propio en esta etapa)

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 8
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 8 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (orquesta la liberación), Banco G08 (dueño del hold que se libera).
*   **Módulos afectados:** Órdenes.
*   **Otros equipos / aprobaciones:** depende del contrato `ITEM_PROVISION_REQUESTED`/`ITEM_PROVISIONED`/`ITEM_PROVISION_FAILED` con Grupo 12 (el mismo que bloquea #138 y #584).
*   **Impacto en datos / migraciones:** ya no hace falta un estado de compensación de saldo propio — se reutiliza la máquina de estados de `Orden` (`HOLD_CONFIRMADO_BANCO → CANCELADA`, CONTEXTO §6); se guarda el motivo de la liberación.
*   **Riesgos y mitigación (opcional):** que esta historia se postergue por ser "el camino de error". Mitigación: no se puede cerrar "Comprar un ítem" (#138) sin los escenarios de falla de esta historia.

