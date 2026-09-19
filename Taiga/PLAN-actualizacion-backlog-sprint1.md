# Plan de actualización del backlog de Taiga (Grupo 11 — Mercado)

> Basado en `docs/CONTEXTO-MERCADO-SPRINT1.md` (14 decisiones confirmadas) y en la auditoría del 18/09/2026 sobre las 6 épicas / 35 historias exportadas en `docs/Taiga/`. Este documento es el plan — todavía no se aplicó nada en Taiga (el acceso por API sigue bloqueado por el 405 de `tree.taiga.io`; hasta que se resuelva, esto se vuelca a mano en la UI).

## Resumen ejecutivo

- **No hacen falta épicas nuevas.** Las 3 épicas que se quedan (#90 Catálogo, #137 Compra Directa, #577 Subastas) ya cubren el alcance confirmado. Se agregan historias y tareas nuevas *dentro* de esas 3, no épicas nuevas.
- **3 épicas se transfieren o se sacan del proyecto de Grupo 11**: #131 (Inventario) entera a Grupo 12; #482 (Boost XP) entera, fuera de Mercado por completo (probablemente mal cargada); #770 (Vencimiento) se divide.
- Los cambios de contenido más urgentes son los que reproducirían bugs ya corregidos en el documento de contexto: **#584** (orden de liquidación invertido) y **#141** (premisa de reembolso que ya no aplica).

---

## Épica #90 — Gestión del Catálogo de Mercado (SE QUEDA, reescritura necesaria)

**Cambio a nivel épica:** sacar la frase "el PROFESOR no interviene en el catálogo — la curaduría por cohorte queda para una iteración posterior". Con la decisión #14, la curaduría del profesor (elegir plantilla + configurar parámetros) **es** el entregable de Sprint 1, no una iteración futura. También sacar cualquier mención a "ADMIN carga ítems concretos" como modelo único.

| US | Cambio |
|---|---|
| **#92 — Crear un ítem** | Reescribir completa: ya no es "ADMIN crea un ítem concreto (nombre/tipo/precio/imagen)". Pasa a ser "PROFESOR configura una oferta de catálogo eligiendo una plantilla (`ItemTemplate`: SHIELD/BOOST_XP/BOOST_COINS/LIFE) y definiendo `precioMonedas` (dentro del rango de PAR-06/07), magnitud y cargas". Agregar body de request/response, y la validación cruzada contra Backoffice (endpoint sincrónico de la decisión #9). |
| **#94 — Consultar catálogo disponible** | Estandarizar `courseCohortId` → `courseId` (convención de plataforma). Agregar el JSON de respuesta completo (lista de `OfertaCatalogo` con nombre derivado de template+config, precio vigente). Citar el contrato real de Cursos que el mock reemplaza (`GET /api/v1/courses/{courseId}/students/{studentId}/enrollment-status`). |
| **#95 — Modificar un ítem** | Reescribir: ya no edita un ítem concreto, edita la configuración de una `OfertaCatalogo` existente (precio, magnitud, cargas, estado) — nunca el tipo de plantilla en sí. |
| **#96 — Activar/desactivar un ítem** | Sin conflicto de fondo. Agregar body de request/response (`{"estado":"ACTIVE"}`). |
| **#98 — Consultar detalle de un artículo** | Sin conflicto. Agregar JSON de respuesta. |

**Historia nueva a agregar:**
- **US nueva — "Consultar los tipos de plantilla disponibles"**: `GET /api/v1/market/templates`, para que el profesor sepa qué tipos existen (SHIELD/BOOST_XP/BOOST_COINS/LIFE) antes de configurar una oferta. Hoy esto está implícito en la decisión #14 pero no tiene historia propia.

**Tareas nuevas a agregar (transversales a la épica):**
- Modelar `ItemTemplate` (tipos cerrados + parámetros configurables) y `OfertaCatalogo` (sin campo de stock — decisión #6).
- Documentar el catálogo de **verbos de efecto** que usa cada plantilla (`ABSORB_FAILURE`, `XP_MULTIPLIER`, `COIN_MULTIPLIER` — nombres en inglés, decisión #4) — esto es lo único que sobrevive de la Épica #131 que se transfiere (ver más abajo); Grupo 12 y Tema 10 lo consumen para aplicar el efecto real.
- Vincular cada historia de esta épica a `CONTEXTO-MERCADO-SPRINT1.md` §5 y §11, y al diagrama de clases corregido en `diagramas-mercado.md`.

---

## Épica #137 — Compra Directa de Ítems del Mercado (SE QUEDA, reescritura necesaria)

| US | Cambio |
|---|---|
| **#138 — Comprar un ítem del catálogo** | Reescribir el CA1 y el BDD alrededor de la saga de 3 pasos: `HOLD_CREATE_REQUESTED`→`HOLD_CREATED` → `ITEM_PROVISION_REQUESTED`→`ITEM_PROVISIONED`/`ITEM_PROVISION_FAILED` (contra Grupo 12, ya no "recibe el ítem en su inventario" local) → `HOLD_CONFIRM_REQUESTED`→`HOLD_CONFIRMED`. Agregar los 3 payloads completos (CONTEXTO §7). |
| **#139 — No pagar dos veces por doble clic** | Sin conflicto. Agregar nombre del header de idempotencia y el código de estado en caso de colisión (409). |
| **#140 — Ver el resultado de una compra en proceso** | **No cerrar como lista todavía**: depende de una pregunta abierta en CONTEXTO §12-C (canal de notificación: ¿polling o push/SSE?). Recomendación: adoptar SSE siguiendo el patrón ya maduro de `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md` (`GET /api/v1/market/orders/stream/{orderId}`), pero esto hay que confirmarlo con el equipo antes de darlo por definitivo. |
| **#141 — Recuperar mis monedas si la compra no se completó** | **Reescritura de la premisa completa.** Ya no es "se cobró pero no se acreditó el ítem → devolver". Con la decisión #8, eso no puede pasar en el camino normal: el hold se libera **antes** de confirmar cualquier débito si la acreditación del ítem falla. Nueva premisa: "si Grupo 12 no puede acreditar el ítem, el hold se libera sin haberse confirmado nunca un débito — no hay nada que reembolsar porque nunca se cobró". |
| **#142 — Comprar una vida sin pasarme del tope** | Sin conflicto (alineado con PAR-12). Agregar contrato. |
| **#143 — Revisar las compras que quedaron a medias** | Sin conflicto. Agregar JSON de respuesta. |

**Historia/tarea nueva a agregar:**
- **Tarea — "Formalizar el contrato `ITEM_PROVISION_REQUESTED`/`ITEM_PROVISIONED`/`ITEM_PROVISION_FAILED` con Grupo 12"**: es un contrato nuevo (CONTEXTO §7/§8.5) que hoy no está acordado bilateralmente — bloquea #138 y #584 hasta que Grupo 12 lo confirme.
- Vincular cada historia a `CONTEXTO-MERCADO-SPRINT1.md` §7 y a la máquina de estados de `Orden` (§6).

---

## Épica #577 — Subastas de Ítems con Tiempo Límite (SE QUEDA, ajustes puntuales)

Es la épica mejor alineada; los cambios son específicos, no una reescritura completa.

| US | Cambio |
|---|---|
| **#578 — Lanzar un ítem a subasta** | Agregar contrato completo; vincular a la máquina de estados de `Subasta` (§6). |
| **#579 — Ver las subastas abiertas de mi curso** | Agregar JSON de respuesta (ya usa `courseId`, correcto). |
| **#580 — Hacer una oferta** | Ya referencia `HOLD_CREATE_REQUESTED`/`HOLD_CREATED` correctamente — agregar los campos del payload (`commandId, auctionId, studentId, amount, orderType:"AUCTION_BID", ttlSeconds, graceBufferSeconds`). |
| **#581 — Mejorar mi oferta** | Ya referencia `HOLD_INCREASE_REQUESTED`/`HOLD_INCREASED` — agregar payload (`currentAmount, newTotalAmount, incrementalAmount`). |
| **#582 — Enterarme al instante si me superaron** | **Renombrar evento**: `OFERTA_SUPERADA` → `BID_OUTBID` (decisión #4). |
| **#584 — Entregar el ítem al ganador al cerrar la subasta** | **La corrección más importante del plan.** (1) Renombrar `SUBASTA_ADJUDICADA` → `AUCTION_AWARDED`. (2) Invertir el orden: corroborar que Grupo 12 puede acreditar el ítem **antes** de confirmar el débito (`CREDITING_ITEM` antes de `CONFIRMING_LEDGER`, §6) — tal como está redactada hoy, reproduce el bug que la auditoría ya encontró y corrigió en `02-matriz-fallos-resiliencia-y-soluciones.md`. (3) Reemplazar "pone el ítem en el inventario del ganador" por la llamada `ITEM_PROVISION_REQUESTED` a Grupo 12. |
| **#585 — Devolver las monedas a quienes no ganaron** | Sin conflicto de nombres. Especificar que el release en batch (`AUCTION_CLOSED{holdIds}`) es obligatorio, no opcional ("de ser posible" → quitar esa condicionalidad). |
| **#586 — Cerrar una subasta sin ofertas** | Reemplazar el campo `result: DESERTED` por el estado canónico `MARKED_DESERTED` dentro del enum de `Subasta` (§6). |
| **#587 — Cancelar una subasta** | Agregar que debe emitir `AUCTION_CANCELLED` (§10) — hoy no lo menciona. |
| **#588 — Cerrar subastas si el curso se archiva o el alumno se da de baja** | **Renombrar eventos**: `CURSO_ARCHIVADO`/`ALUMNO_DESMATRICULADO` → `COURSE_ARCHIVED`/`STUDENT_UNENROLLED`, tópico `courses.lifecycle` (§9.2/§10). |
| **#589 — Terminar cierres de subasta a medias** | Estandarizar nombre de tabla: `outbox_event` → `outbox_events` (§8.3/§8.6). |

**Historia/tarea nueva a agregar:**
- **Tarea — "Manejar `HOLD_REJECTED` al pujar"**: hoy ninguna historia cubre el caso de que Banco rechace el hold de una puja (evento ya definido en CONTEXTO §8.5, pero sin historia que lo consuma).
- Vincular cada historia a `CONTEXTO-MERCADO-SPRINT1.md` §8 y a la máquina de estados de `Subasta`/`Puja` (§6).

---

## Épica #131 — Inventario, Equipamiento y Consumo del Alumno → SE TRANSFIERE a Grupo 12

Acción: mover la épica completa (#132, #133, #134, #135, #136) al proyecto de Grupo 12 en Taiga. No se reescribe contenido — es responsabilidad de Grupo 12 adaptarlo a su propio modelo.

**Lo único que Mercado se queda:** el vocabulario de verbos de efecto que hoy vive en los supuestos de esta épica y en #135 (`ABSORBER_FALLO`, `MULTIPLICAR_XP`) — pasa a la Épica #90 como la tarea "documentar catálogo de verbos de efecto", renombrado a inglés (`ABSORB_FAILURE`, `XP_MULTIPLIER`, `COIN_MULTIPLIER`).

## Épica #482 — Otorgamiento de Boost de XP → SE SACA de Mercado (probable error de carga)

Acción: no tiene ninguna mención a Mercado, ítems, catálogo ni monedas — es territorio de Motor de Desafíos/Roadmap. Recomendación: reportarlo al PO/admin de Taiga para reasignar el proyecto correcto. De paso, **cerrar #810 como duplicado de #130** (contenido idéntico, distinto ID y propietario — bug de exportación/clonado en Taiga).

## Épica #770 — Vencimiento de Ítems del Inventario → SE DIVIDE

| US | Destino |
|---|---|
| **#778 — Configurar el vencimiento de un ítem del curso** | Se queda en Mercado — pasa a ser un campo de configuración dentro de `OfertaCatalogo.configuracion` (Épica #90). Nota: la regla exacta (¿por fecha, por cierre de curso, o ambas?) sigue abierta en CONTEXTO §12-C — no cerrarla sin confirmar con el equipo. |
| **#785 — Ver cuándo vencen mis ítems** | Se divide: la parte "cuándo vence este tipo de oferta" se queda en catálogo; la parte "fecha exacta de mi instancia" se transfiere a Grupo 12. |
| **#786, #787, #788** | Se transfieren enteras a Grupo 12 (operan sobre la instancia de `ItemInventario`, ya no es de Mercado). Al transferir, renombrar sus eventos (`ITEM_EXPIRING_SOON`, `ITEM_EXPIRED`) a la convención en inglés que ya usa Mercado — pasan a ser responsabilidad de Grupo 12 como dueño nuevo. |

---

## Punch list de ejecución (orden recomendado)

1. Reescribir #584 (orden + eventos) y #141 (premisa) — son las que reproducirían bugs ya corregidos.
2. Reescribir #92/#95 y la épica #90 al modelo de plantillas (decisión #14) — bloquea todo lo que depende del catálogo.
3. Renombrar eventos sueltos en español: #582, #588.
4. Completar contratos (request/response JSON) en las 23 historias que se quedan, usando CONTEXTO §7/§8.5/§9 como fuente.
5. Agregar las 3 historias/tareas nuevas (tipos de plantilla, contrato `ITEM_PROVISION_*` con Grupo 12, manejo de `HOLD_REJECTED`).
6. Vincular cada historia a la sección correspondiente de `CONTEXTO-MERCADO-SPRINT1.md` y a `diagramas-mercado.md`.
7. Coordinar con Grupo 12: transferir Épica #131 completa + #786/#787/#788 de la #770, con el hand-off de verbos de efecto.
8. Reportar al PO/admin de Taiga: reasignar Épica #482 fuera de Mercado, y cerrar #810 como duplicado de #130.
9. Resolver con el equipo la pregunta abierta de #140 (polling vs. SSE) antes de darla por dev-ready.
10. Confirmar con el equipo la regla de vencimiento de #778 (fecha / cierre de curso / ambas) antes de cerrarla.
