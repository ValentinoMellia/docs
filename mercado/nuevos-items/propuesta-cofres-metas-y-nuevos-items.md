# Propuesta: Cofres, Metas colectivas y nuevos ítems

> **Estado: propuesta para discutir en grupo (29/09/2026).** Nada de esto está implementado ni acordado con el PO. Las decisiones abiertas están marcadas como **[D-n]** al final; llevarlas al [taller de decisiones](../estado-actual/taller-decisiones.html).
> Base: [`gestion-de-tienda.md`](../estado-actual/gestion-de-tienda.md), [`compra.md`](../estado-actual/compra.md), [`subasta.md`](../estado-actual/subasta.md) y las reglas del PRD resumidas en [`CONTEXTO-MERCADO-SPRINT1.md` §4](../../arquitectura/CONTEXTO-MERCADO-SPRINT1.md).

## 0. Criterios para aceptar un ítem nuevo

Un ítem entra al catálogo solo si cumple todo esto:

1. **Encaja en lo que ya existe.** Se publica como oferta sobre una plantilla (decisión #14), se compra con la saga hold → acreditar → cobrar, y no pide topics ni servicios nuevos salvo que se justifique.
2. **Efecto acotado.** Tiene tope numérico (cargas, minutos, cantidad). Nada permanente ni ilimitado.
3. **Solo afecta a quien lo usa.** No hay ítems que perjudiquen a otro alumno ni que transfieran monedas entre alumnos.
4. **No convierte monedas en ranking.** Comprar no debería dar ventaja directa en el ranking (XP) sin haber resuelto desafíos.
5. **Respeta dueños.** Mercado no decide sobre vidas, XP ni monedas ajenas (Tema 10 y Accounting); solo define el ítem y pide la acreditación.
6. **Todo por curso** (RF-REC-01, RF-INT-04).

## 1. Dos reglas del PRD que chocan con la idea tal como está

| Regla | Qué dice | Qué afecta |
|---|---|---|
| **RF-INT-01** | Las monedas solo se canjean por **vidas o equipamiento**; **nunca se obtienen monedas por intercambio**. | Un cofre que devuelve **monedas** la viola de forma literal. Un cofre que da **XP** también se sale de "vidas o equipamiento". |
| **RF-INT-03** | Hay **dos** modalidades: compra directa y subasta. | La **meta colectiva** es una tercera modalidad. |

Estas reglas figuran como "no sujetas a replanteo salvo consenso con el PO". Hay dos caminos:

- **Camino A (sin tocar el PRD):** cofre que solo da **ítems** (equipamiento). La meta colectiva se presenta como una variante de compra directa: se paga el precio entre varios en lugar de uno solo. Hay que confirmarlo igual con el PO, pero el cambio de regla es menor.
- **Camino B (pedir el cambio al PO):** cofre con monedas y XP. Se aceptan las salvaguardas de §2.4 como condición.

**Recomendación:** implementar el cofre con ítems primero (Camino A) y dejar monedas y XP como extensión **apagada por defecto** (feature toggle) mientras se consulta al PO. El modelo de §2.2 ya los contempla, así que después solo hay que encenderlos. → **[D-1]**

## 2. Cofre (`CHEST`)

### 2.1 Qué es

Una nueva plantilla. El profesor publica ofertas de tipo `CHEST` y cada una es un "nivel" de cofre (por ejemplo Bronce, Plata, Oro). Mantenemos la doctrina de "sin tiers fijos": el nivel es solo el nombre y la configuración que elige el profesor. Mercado no trae cofres prearmados.

Al comprarlo, el alumno recibe:

- **Ítems**: una cantidad al azar dentro de un rango, sorteados de un pool que arma el profesor.
- **Monedas**: un valor al azar dentro de `[min, max]` (solo si se habilita, [D-1]).
- **XP**: un valor al azar dentro de `[min, max]` (solo si se habilita, [D-1]).

### 2.2 Parámetros que configura el profesor

| Parámetro | Tipo | Validación | Ejemplo (Plata) |
|---|---|---|---|
| `coinPrice` | entero | > 0 | 800 |
| `tierLabel` | texto | ≤ 30 caracteres; solo se muestra | "Plata" |
| `itemRolls` | `{min, max}` | 1 ≤ min ≤ max ≤ **5** | `{1, 2}` |
| `itemPool[]` | lista | 1 a **8** entradas | ver abajo |
| `itemPool[].itemConfig` | configuración embebida de un `SHIELD` / `BOOST_XP` / `BOOST_COINS` | la misma validación que al publicar ese tipo | escudo de 1 carga |
| `itemPool[].weight` | entero | 1 a 100 | 60 |
| `itemPool[].maxPerChest` | entero | ≥ 1 | 1 |
| `coinReward` | `{min, max}` \| `null` | 0 ≤ min ≤ max **< `coinPrice`** | `{100, 400}` |
| `xpReward` | `{min, max}` \| `null` | 0 ≤ min ≤ max ≤ `PAR-XP-CHEST-MAX` (tope de ADMIN) | `{20, 60}` |
| `totalStock`, `publicationTtlMinutes` | como en cualquier oferta | ya existen | `null`, 10 080 |
| `maxPerStudent` | entero \| `null` | ≥ 1 | 3 |

Criterios de diseño:

- **El pool embebe la configuración del ítem** en lugar de apuntar a otra oferta. Así, si el profesor desactiva o edita una oferta, los cofres ya publicados no cambian. Es coherente con la regla actual de que la configuración de una oferta es inmutable después de publicarla.
- **Sin vidas en el pool** (al menos al principio): una vida sorteada puede superar el tope de PAR-12, y el rechazo por tope (US-142) todavía no funciona. → **[D-3]**
- **Sin cofres dentro de cofres:** así no hay recursión.
- **Transparencia:** la vitrina muestra la probabilidad de cada ítem (`weight / suma`) y los rangos. Es un contexto educativo con alumnos, así que no ocultamos cómo funciona el azar.

### 2.3 Cómo se abre (flujo)

**Recomendación: la compra abre el cofre.** El cofre no entra a la mochila cerrado. Si entrara cerrado, Accounting o Inventario tendrían que llamar a Mercado para abrirlo, y hoy no existe ese camino.

1. `POST /courses/{courseId}/orders` con la oferta del cofre, igual que cualquier compra (idempotencia, stock y vencimiento no cambian).
2. **Sorteo al crear la orden**, en la misma transacción: el resultado (ítems, monedas, XP) se guarda en la orden (`orders.chest_result`, JSON) junto con la semilla. Si hay un reintento o llega un mensaje duplicado, **no se vuelve a sortear**.
3. Hold del precio (`DIRECT_PURCHASE`), igual que hoy.
4. Acreditación de **N ítems**. Hoy la saga acredita un solo ítem por orden. Hay dos alternativas → **[D-2]**:
   - a) Un solo comando con una lista de ítems (Accounting tiene que aceptar listas).
   - b) Un comando por ítem, con `commandId` derivado (`orderId:1`, `orderId:2`…). Mercado espera todas las confirmaciones antes de cobrar y, si alguna falla, libera el hold. Queda un caso delicado: si algunos ítems ya se acreditaron, no hay forma de devolverlos, porque Accounting no tiene "quitar ítem". Por eso (a) es más segura.
5. Cobro del hold (`HOLD_CONFIRM_REQUESTED`).
6. **Después del cobro**, y solo si están habilitados: acreditar las monedas (a Accounting, un comando de tipo `CHEST_REWARD_CREDIT`) y el XP (a Tema 10). Van al final porque son "regalos": si fallan se reintentan (con outbox), pero no deshacen la compra.

La respuesta de la orden (`GET /orders/{id}` y SSE) incluye `chestResult`, para que el front muestre la animación de apertura.

### 2.4 Salvaguardas si se habilitan monedas y XP

- **`coinReward.max < coinPrice`**: el cofre nunca devuelve más monedas de las que cuesta. Si no, se puede "farmear" comprando cofres en loop.
- **Valor esperado visible**: al publicar, el formulario muestra el promedio de monedas devueltas y advierte si supera el 60 % del precio.
- **`maxPerStudent`** (por ejemplo por semana, **[D-4]**) para que el XP no se pueda comprar sin límite.
- **Tope global de XP por cofre** (`PAR-XP-CHEST-MAX`) definido por ADMIN, no por el profesor. Evita que un curso regale XP a montones y distorsione el ranking.
- El XP del cofre se acredita como **XP de recompensa**, no de desafío. Así no activa `BOOST_XP` (los boosts no se multiplican entre sí).

### 2.5 Impacto por servicio

| Servicio | Cambio |
|---|---|
| Mercado | Nueva plantilla `CHEST` y su validador; sorteo y `chest_result` en la orden; saga con N ítems; outbox para monedas y XP |
| Accounting | Acreditación de varios ítems ([D-2]); si se habilitan monedas, un crédito de monedas que no sea compra ni subasta |
| Tema 10 | Si se habilita XP, un comando o evento de "otorgar XP de recompensa" |
| Front | Formulario del pool con probabilidades; animación de apertura |

## 3. Meta colectiva (`GOAL`, "Colecta")

> **Se movió a su propio documento:** [`../metas-colectivas/README.md`](../metas-colectivas/README.md). Ese documento manda.

Resumen: el profesor define el ítem (a partir de una oferta del curso o desde una plantilla), el monto y el cierre. Si se llega al monto, todos los que aportaron reciben el ítem. Si no, pierden lo aportado (`forfeitPercent`, 100 % por defecto). Cómo se acredita el mismo ítem a varios alumnos queda **pendiente de Banco**.

## 4. Otros ítems propuestos

Ninguno afecta a otros alumnos, todos tienen tope y ninguno se usa en exámenes. En "Depende de", **solo Mercado** significa que no hace falta otro equipo más allá del crédito de ítem que ya existe.

### 4.1 Ayuda para aprender (el sumidero de monedas más útil)

Son los más valiosos para la plataforma: el alumno gasta monedas en **aprender mejor**, no en saltearse el aprendizaje.

| Ítem (plantilla) | Qué hace | Parámetros | Depende de | Riesgo |
|---|---|---|---|---|
| **Pista** (`HINT`) | Muestra la pista que el profesor cargó en un desafío | `coinPrice`, `charges`, `applicableChallenges` (sin `ALL`) | Desafíos (Tema 03): el profesor carga pistas | Bajo. No revela la respuesta |
| **Consultas extra al tutor IA** (`AI_ASSIST`) | +N consultas al agente de IA cuando el alumno agotó las del día. El agente sigue con sus restricciones pedagógicas (nunca da la solución) | `coinPrice`, `extraQueries` (1 a 10), `validForHours` | Tema de agentes IA | Bajo. Solo aplica si el agente tiene cupo diario |
| **50/50** (`FIFTY_FIFTY`) | En una pregunta de opción múltiple, descarta la mitad de las opciones incorrectas | `coinPrice`, `charges` (1 a 3); solo `THEORETICAL_ONLY` | Desafíos | Bajo |
| **Revelar un caso de prueba** (`TEST_REVEAL`) | En un desafío práctico, muestra la entrada y la salida esperada de un test oculto que está fallando | `coinPrice`, `charges`; solo `PRACTICAL_ONLY` | Desafíos | Bajo o medio: el profesor tiene que marcar qué tests se pueden revelar |
| **Desbloquear solución de referencia** (`SOLUTION_UNLOCK`) | Después de **aprobar** un desafío, muestra la solución del profesor para comparar | `coinPrice`, `charges` | Desafíos | Muy bajo: solo se usa con el desafío aprobado |
| **Desafío bonus** (`BONUS_CHALLENGE`) | Desbloquea un desafío opcional extra (práctica adicional) que da XP normal al resolverlo | `coinPrice`, `challengePoolId` | Desafíos + Roadmap | Bajo. El XP se gana resolviendo, no comprando |

### 4.2 Protección y tiempo

| Ítem (plantilla) | Qué hace | Parámetros | Depende de | Riesgo |
|---|---|---|---|---|
| **Protector de racha** (`STREAK_FREEZE`) | Si el alumno no resuelve nada un día, la racha no se corta | `coinPrice`, `charges` (1 a 3), máximo 1 equipado | Tema 10 (dueño de la racha) | Bajo |
| **Cambio de desafío de recuperación** (`RECOVERY_REROLL`) | Con 0 vidas, cambia el desafío de recuperación asignado por otro del pool (RF-REC-06) | `coinPrice`, `charges` (1) | Desafíos | Bajo. Útil cuando el alumno se trabó con uno |
| **Tiempo extra** (`EXTRA_TIME`) | +N minutos en un desafío con tiempo | `coinPrice`, `extraMinutes` (5 a 30) | Desafíos | Medio: depende de que haya desafíos con tiempo |
| **Pase de entrega tardía** (`LATE_PASS`) | Extiende el plazo de una entrega práctica | `coinPrice`, `extraHours` (≤ 48) | Desafíos / Cursos | Medio: el profesor tiene que poder deshabilitarlo por desafío |

### 4.3 Mercado y metas (se combinan con la meta colectiva y la subasta)

| Ítem (plantilla) | Qué hace | Parámetros | Depende de | Riesgo |
|---|---|---|---|---|
| **Seguro de colecta** (`GOAL_INSURANCE`) | Se equipa antes de aportar a una meta. Si la meta falla, recupera un % de lo que ese alumno perdió. Se consume solo si la meta falla | `coinPrice`, `refundPercent` (10 a 50), `charges` (1) | Solo Mercado + la captura parcial de [D-10] | Bajo. Suaviza la regla de pérdida sin eliminarla |
| **Cofre de meta** | No es una plantilla nueva: el premio de una meta colectiva puede ser un `CHEST` en lugar de un ítem fijo; cada aportante abre el suyo | el `CHEST` del §2 | Solo Mercado | Bajo. Hace más atractiva la meta |
| **Aviso de subasta** (`AUCTION_ALERT`) | Notifica al alumno cuando se publica una subasta o meta del tipo de ítem que elija | `coinPrice`, `itemTypeFilter`, `validDays` | Notificaciones (Tema 11) | Muy bajo |

### 4.4 Social y cosmético (sin efecto en el juego)

| Ítem (plantilla) | Qué hace | Parámetros | Depende de | Riesgo |
|---|---|---|---|---|
| **Cosmético** (`COSMETIC`) | Marco de avatar, título o color del nombre en el ranking | `coinPrice`, `cosmeticKind` (`FRAME` \| `TITLE` \| `NAME_COLOR`), `assetKey` | Front + perfil (Tema 10) | Muy bajo |
| **Reconocimiento** (`KUDOS`) | Se le envía a un compañero un reconocimiento ("¡Gracias por la ayuda!") que queda visible en su perfil. No tiene valor: no se puede vender ni usar | `coinPrice`, `maxPerWeek` (1 a 3), mensajes predefinidos (no texto libre) | Perfil (Tema 10) | Bajo. Premia la colaboración. Mensajes cerrados para evitar abusos |

Aclaración para cosméticos y reconocimientos: RF-INT-01 habla de "vidas o equipamiento". Hay que confirmar con el PO que entran como canje válido (junto con D-1).

### 4.5 Por dónde empezar

| Prioridad | Ítems | Por qué |
|---|---|---|
| 1 | `CHEST` (solo ítems), `COSMETIC` | Solo Mercado + front |
| 2 | Meta colectiva + `GOAL_INSURANCE` + cofre de meta | Después de la subasta, sobre el mismo núcleo |
| 3 | `HINT`, `SOLUTION_UNLOCK`, `FIFTY_FIFTY`, `STREAK_FREEZE`, `KUDOS` | Una sola dependencia externa cada uno; alto valor pedagógico |
| 4 | `AI_ASSIST`, `TEST_REVEAL`, `BONUS_CHALLENGE`, `RECOVERY_REROLL`, `EXTRA_TIME`, `LATE_PASS`, `AUCTION_ALERT` | Necesitan acuerdos con otros equipos |

### 4.6 Ítems que conviene no hacer

| Idea | Por qué no |
|---|---|
| Comprar XP directo | Convierte monedas en ranking (criterio 4) |
| Ítems contra otro alumno (robar monedas, "congelar" a un compañero) | Rompe el criterio 3 y genera conflicto en el aula |
| Regalar o transferir monedas entre alumnos | Viola RF-INT-01 y RF-INT-02 |
| Multiplicadores permanentes o acumulables | Sin tope; con dos boosts ×2 activos se llega a ×4. Regla sugerida: **un solo boost activo por tipo** ([D-8]) |
| Saltear un desafío / aprobarlo automáticamente | Rompe el objetivo pedagógico |
| Apuestas ("doble o nada", apostar monedas al resultado de un desafío) | Es azar puro sin ítem a cambio y premia arriesgar, no aprender. El cofre ya cubre la parte de sorpresa con reglas acotadas |

## 5. Decisiones abiertas

Las decisiones de la meta colectiva (D-5, D-6, D-7, D-9 y D-10) se siguen en [`../metas-colectivas/README.md` §11](../metas-colectivas/README.md) (M-1 a M-7).

| Id | Pregunta | Recomendación |
|---|---|---|
| **D-1** | ¿El cofre da monedas y XP (cambia RF-INT-01) o solo ítems? | Solo ítems primero; monedas y XP detrás de un toggle, pendiente del PO |
| **D-2** | ¿Cómo se acreditan N ítems de una orden? | Un comando con lista (pedido a Accounting) |
| **D-3** | ¿Se permiten vidas en el pool del cofre o como premio de una meta? | No, hasta que funcione el tope de vidas (US-142) |
| **D-4** | Período de `maxPerStudent` del cofre | Por semana |
| **D-5** | ¿Se puede retirar un aporte a una meta abierta? | No (igual que la subasta). Con la pérdida por fallo esto pesa más: si se pudiera retirar, todos esperarían al último minuto y la regla perdería sentido | 
| **D-6** | Máximo de metas abiertas por curso | 3 | 
| **D-7** | `orderType` del aporte | `GOAL_CONTRIBUTION` nuevo en Accounting | 
| **D-8** | ¿Los boosts del mismo tipo se acumulan? | No: un solo boost activo por tipo |
| **D-9** | ¿La meta colectiva necesita el visto bueno del PO por RF-INT-03? | Sí, consultarlo junto con D-1 | 
| **D-10** | Si la meta falla, ¿se pierde siempre todo o el profesor elige el % (`forfeitPercent`)? El % intermedio necesita captura parcial en Accounting | Configurable, 100 % por defecto; en el MVP solo 0 o 100 si Accounting no suma captura parcial | 
| **D-11** | ¿Cosméticos y reconocimientos cuentan como canje válido según RF-INT-01? | Consultar al PO junto con D-1 |
