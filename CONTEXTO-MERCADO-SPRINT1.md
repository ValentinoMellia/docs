# Contexto consolidado — Mercado (Tema 09) para Sprint 1

> ⚠️ **Estado: borrador para revisión, no es la versión final.** Este documento centraliza el contexto disponible al 16/09/2026. El equipo va a sumar definiciones nuevas del PO en las próximas horas que todavía no están escritas en ningún archivo — principalmente sobre el **alcance real de Sprint 1**, el **mecanismo de consumo de equipamiento** (quién invoca a quién), la **sincronización de PAR-06/07 vía Gateway**, y la **desmatriculación a mitad de curso** (ver detalle completo en la Sección 12-B). Hay que retocar este archivo apenas lleguen esas definiciones — no darlo por cerrado todavía.

> **Alcance de este documento:** mercado, subastas, compra directa e inventario — el foco de trabajo actual del equipo. No es un resumen: es un análisis exhaustivo pensado para que el agente que arranca el ciclo SDD (`sdd-explore` → `sdd-propose` → `sdd-spec` → `sdd-design` → `sdd-tasks` → `sdd-apply`) tenga todo el contexto técnico y de negocio sin tener que reconstruirlo leyendo 10 documentos dispersos.

---

## 1. Propósito y cómo leer este documento

Este archivo es un **mapa de navegación con profundidad técnica real**, no un índice ni un resumen ejecutivo. Cita nombres de eventos, campos de payloads, IDs de requerimiento (`RF-XXX`, `PAR-XXX`) y fragmentos de contratos tal como aparecen en las fuentes, para que las decisiones de diseño del Sprint 1 se puedan tomar sin adivinar nada.

**Qué NO es:** no reemplaza a los documentos fuente (ver [Sección 13 — Referencias](#13-referencias)). Cuando haga falta el detalle extremo — el diagrama Mermaid completo, el JSON exacto de un contrato, el mockup — hay que ir a la fuente original. Este documento da la síntesis con trazabilidad suficiente para saber *dónde* mirar y *qué* está resuelto.

**Advertencia importante — léela antes de seguir:** buena parte de las ambigüedades y contradicciones que había entre los documentos fuente **se resolvieron en una conversación directa con el equipo mientras se armaba este documento** (16/09/2026). Esas resoluciones **todavía no están volcadas en los documentos fuente originales** (`mercado-kickoff (1).md`, `diagramas-mercado.md`, `Mercado/Subastas/*`) — se van a actualizar en las próximas horas. Este documento **sí** las incorpora. Si en algún punto este documento contradice a un doc fuente, **este documento manda** para los 8 puntos listados en la [Sección 12-A](#12-a-decisiones-confirmadas-en-esta-sesión); para todo lo demás, la fuente original sigue siendo la autoridad.

### TL;DR de las decisiones ya tomadas (detalle completo en Sección 12)

1. Subastas = **Fase 3** (confirmado; la mención de "Fase 2" en el spike M-00 es un error de redacción).
2. Broker de mensajería = **Kafka**, definitivo para toda la plataforma.
3. Modelo de holds en subastas = **Opción 1 (Hold Escrow Total)**, confirmada como arquitectura a implementar (no como paso intermedio).
4. Nomenclatura de eventos de Mercado = **MAYÚSCULAS_SNAKE_CASE, en inglés**, sin excepción.
5. El equipamiento con efecto mecánico lo define **el profesor, por curso-cohorte**.
6. **Sin límite de stock** en la oferta de catálogo.
7. Cursos (Tema 02) debe **bloquear el archivado** de un curso mientras tenga subastas activas.
8. Validaciones de negocio (ej. tope de vidas) se resuelven **reservando → corroborando la acreditación → confirmando el débito recién si la acreditación se corroboró**. No hace falta compensación para ese caso.

Y 4 pendientes genuinos que siguen sin resolver — no los dés por definidos: sincronización de PAR-06/07 vía Gateway, desmatriculación a mitad de curso, quién invoca a quién para consumir equipamiento, y **el alcance real de Sprint 1 en sí**.

---

## 2. El sistema completo (Aula Quest) y el lugar de Mercado en él

**Aula Quest** es una plataforma de e-learning gamificada — Trabajo Práctico Integrador de Programación IV Back End, TUP UTN FRC. Se construye como un ecosistema de microservicios repartido en **12 temas/grupos**, cada uno dueño exclusivo de su base de datos y de un dominio.

Cada curso es un roadmap de aprendizaje incremental. El alumno resuelve desafíos, acumula XP, monedas, insignias, vidas y equipamiento, compite en un ranking por curso, y es asistido (nunca reemplazado) por agentes de IA con restricciones pedagógicas estrictas.

### Reglas de plataforma no negociables (aplican a todos los temas, incluido Mercado)

1. El **API Gateway (Tema 01)** es la única puerta de entrada. Nada de comunicación directa entre microservicios.
2. Cada servicio es **dueño exclusivo de su base**. Nadie lee la tabla del vecino.
3. Todo lo sincrónico sale y entra por el Gateway; todo lo asincrónico va por el **bus de eventos Kafka** (confirmado, ver decisión #2).
4. **No hay borrado físico** de nada (salvo el chat social) — todo es baja lógica (RF-NFR-01).
5. Cada entidad tiene **un dueño único**.

### Temas relevantes para Mercado

| Tema | Nombre | Relación con Mercado |
|---|---|---|
| **01** | Identidad y Gateway | Perimetral: valida JWT, inyecta `X-User-Id`/`X-Roles`/`X-User-Email`. Mercado nunca ve credenciales, solo cabeceras de confianza. |
| **02** | Cursos y Matrícula | Dueño del curso-cohorte y su ciclo de vida (draft→activo→archivado). Publica eventos de archivado/desmatriculación que Mercado debe procesar. |
| **03** | Motor de Desafíos | Consulta a Mercado qué ítems tiene equipados el alumno antes de correr un desafío; reporta el consumo vía evento asíncrono. |
| **05** | Runner | Sandbox Docker efímero que ejecuta tests para el Motor de Desafíos (colaborador de Tema 03, sin relación directa con Mercado). |
| **08** | Banco | **Dependencia crítica bloqueante.** Dueño exclusivo del ledger y los saldos. Mercado nunca guarda saldo — todo pasa por reserva/confirmación/liberación contra Banco. |
| **09** | **Mercado** | Este tema: catálogo, compra directa, subastas, inventario/mochila del alumno. |
| **10** | Ranking / Roadmap y Progreso | **Dependencia crítica.** Dueño de XP, niveles, vidas, logros e insignias. Mercado vende vidas y equipamiento, pero es Tema 10 quien las acredita/gobierna sus reglas (ej. tope de vidas PAR-12). |
| **11** | Notificaciones (Social) | **Dueño del contrato de eventos de toda la plataforma.** Consume los eventos de Mercado para notificar al alumno. |
| **12** | Backoffice | Administra los parámetros económicos globales (PAR-01 a PAR-24). Mercado consume PAR-06 y PAR-07 (precios) sin poder sobreescribirlos. |

---

## 3. Alcance oficial (PRD) vs. alcance de trabajo del equipo

### 3.1 Fasificación oficial del PRD (fuente de verdad del producto)

| Fase | Contenido relevante a Mercado |
|---|---|
| **MVP** | XP/monedas/vidas básicas, ranking simple — **sin** catálogo de intercambio |
| **Fase 2** | Insignias, **equipamiento**, **sistema de intercambio por compra directa** |
| **Fase 3** | **Subastas**, agentes de IA en chat grupal, modo examen |

El PRD es explícito (Sección 18, Tabla 11): **ni compra directa ni subastas son parte del MVP**. Compra directa es Fase 2, subastas es Fase 3.

### 3.2 Alcance propuesto por la arquitectura general del TPI (kickoff)

| Prioridad | Ítems |
|---|---|
| **Pedido para empezar** | Catálogo · Compra contra reserva del Banco · Inventario del alumno por curso · Consumo de ítems |
| **Para más adelante** | Subastas con ventana temporal · Acceso móvil a subastas · Vencimiento de ítems |
| **Podría ser** | Intercambio entre alumnos · Ítems por temporada · Catálogo configurable por curso |

Nota textual del documento de arquitectura: *"Es el tema más liviano del reparto: los extras son el mecanismo previsto para equilibrarlo."* Con 10-11 personas, el núcleo (catálogo + compra + inventario + consumo) no llena el cuatrimestre — de ahí la sugerencia de tomar trabajo extra (ver Sección 11).

### 3.3 Confirmado en esta sesión

- **Subastas = Fase 3**, sin ambigüedad. El spike M-00 del Sprint 0 menciona "subastas de Fase 2" en una de sus preguntas — es un **error de redacción del equipo**, no un cambio real de alcance (decisión #1).
- El TPI, al no ser el MVP estricto del producto, sí está avanzando sobre alcance de Fase 2 (compra directa, catálogo) en sprints tempranos — esto es una decisión de secuenciación del trabajo práctico, no una contradicción con el PRD.

---

## 4. Reglas de negocio cerradas (fuente: PRD — "no sujetas a replanteo" salvo consenso con el PO)

### 4.1 Sistema de intercambio (Sección 10 del PRD — el núcleo de Mercado)

- **RF-INT-01**: las monedas solo se canjean por **vidas o equipamiento**; nunca al revés; **nunca se obtienen monedas por intercambio** (el intercambio es un sumidero, no una fuente de monedas).
- **RF-INT-02**: solo las monedas son intercambiables (insignias, XP y vidas no se canjean directamente entre sí).
- **RF-INT-03**: dos modalidades — **compra directa** (precio fijo) y **subasta** (mayor oferta se lleva el ítem).
- **RF-INT-04**: las monedas usadas deben pertenecer **al mismo curso** donde se realiza el intercambio.
- **RF-INT-05** — reglas completas de subasta:
  - el PROFESOR lanza el asset y define **duración** y (opcionalmente) **puja mínima** de entrada;
  - al pujar, las monedas quedan **bloqueadas/reservadas** (no disponibles para compra directa ni otras subastas) hasta el cierre;
  - el pujador puede **aumentar** su oferta, **nunca retirarla ni bajarla**;
  - al cierre: al ganador se le descuentan las monedas y recibe el asset; a los demás se les **liberan las reservas sin costo**;
  - subasta sin pujas → asset **sin asignar**.
- **RF-INT-06**: el profesor puede **cancelar** una subasta en curso; nadie recibe el asset y todas las pujas se liberan íntegramente.

### 4.2 Recompensas (Sección 9 del PRD — define qué vende Mercado)

- **RF-REC-01**: las recompensas de un curso solo se usan en ese mismo curso, **sin excepción**.
- **RF-REC-02**: las insignias son cosméticas/prestigio, no se usan ni consumen.
- **RF-REC-03**: insignias = cosmético sin efecto mecánico. **Equipamiento = único tipo de recompensa con efecto mecánico** (ej. escudo que evita perder una vida).
- **RF-REC-04**: "Desafío de recuperación de vida" — solo para alumnos con 0 vidas, no consume vidas, reintentable indefinidamente, obligatorio para seguir tomando desafíos.
- **RF-REC-05**: el equipamiento con efecto mecánico **se consume al usarse** (uso único) → el inventario necesita **estado por instancia**, no un contador.
- **RF-REC-06**: el profesor carga un pool de desafíos de recuperación de vida; el sistema elige uno al azar priorizando los no resueltos.

### 4.3 Configuración — catálogo de parámetros económicos (Sección 4.1 del PRD)

Todo el catálogo de economía es configuración global **exclusiva de ADMIN** (RF-CFG-04/05) — el profesor no puede sobreescribirlo, y Mercado no debe hardcodear ningún valor.

| ID | Parámetro | Default |
|---|---|---|
| **PAR-06** | Precio en monedas de 1 vida | **300** |
| **PAR-07** | Precio de equipamiento con efecto mecánico | **500** |
| **PAR-12** | Vidas iniciales / máximo vigentes por curso | **3 / 3** |

- **RF-CFG-06**: un cambio de parámetro **rige solo hacia adelante** — toda orden de compra debe guardar el **precio con el que se ejecutó** (snapshot), no una referencia al parámetro vigente.

### 4.4 Requerimientos no funcionales relevantes

- **RF-NFR-01**: borrado lógico en todas las entidades de Mercado (catálogo, inventario, órdenes) — sin excepción.
- **RF-NFR-03**: la plataforma soporta 120 usuarios / 120 sesiones concurrentes. Para Mercado, el pico real **no es el catálogo, es el cierre simultáneo de una subasta**.
- **RF-NFR-06 / Tabla 9** (disponibilidad por formato):
  - **Catálogo de intercambio (compra directa)**: solo **Escritorio**.
  - **Subastas: seguimiento y puja**: **Escritorio y Móvil** — motivo explícito del PRD: la subasta tiene ventana de tiempo acotada; un alumno no puede quedar fuera de la competencia por no estar frente a una PC en el momento del cierre.
  - Si el alumno entra desde móvil a una sección no habilitada, la plataforma debe **informarlo explícitamente**, nunca degradarse o fallar en silencio.
- **RF-CUR-08/09**: curso archivado = solo lectura, sin nuevas acciones.
- **RF-NOT-02**: "nuevos intercambios disponibles" es un evento notificable → Mercado debe publicar eventos para Tema 11.
- **RF-TUR-04**: el Guided Tour incluye un tour de canje obligatorio → el catálogo es parte del onboarding de la plataforma.

### 4.5 Gap detectado en el PRD

El registro de riesgos oficial del PRD (Tabla 12, RSK-01 a RSK-14) **no incluye ningún riesgo específico de la economía de intercambio** (fraude en subastas, doble gasto, condiciones de carrera en compra directa). El equipo Mercado ya lo compensa parcialmente exigiendo pruebas de concurrencia en su propia Definition of Done (ver Sección 11), pero es un hueco real del documento madre que vale la pena que el equipo registre por su cuenta.

---

## 5. Modelo de dominio

Entidades candidatas, todas con `cursoCohorteId` obligatorio y baja lógica (RF-NFR-01). Dueño exclusivo: Tema 09.

| Entidad | Atributos clave |
|---|---|
| **ItemDefinicion** | `id`, `nombre`, `tipo` (VIDA \| EQUIPAMIENTO), `efecto` (contrato de nombre con Tema 10), `consumible`, `activo` |
| **OfertaCatalogo** | `id`, `cursoCohorteId`, `itemDefinicionId`, `precioMonedas`, `estado` — **sin campo de stock** (decisión #6: disponibilidad siempre ilimitada mientras esté activa) |
| **Orden** | `id`, `cursoCohorteId`, `alumnoId`, `ofertaId`, `precioAplicado` (**snapshot**, RF-CFG-06), `reservaId`, `idempotencyKey`, `estado`, `creadaEn` |
| **Subasta** | `id`, `cursoCohorteId`, `itemDefinicionId`, `profesorId`, `inicio`, `fin`, `pujaMinima`, `estado`, `version` (bloqueo optimista) |
| **Puja** | `id`, `subastaId`, `alumnoId`, `monto`, `reservaId`, `estado`, `creadaEn` |
| **ItemInventario** | `id`, `cursoCohorteId`, `alumnoId`, `itemDefinicionId`, `origen` (COMPRA \| SUBASTA \| DESAFIO), `estado`, `consumidoEn`, `version` |

**Lo que NO es de Mercado:** saldo de monedas (Tema 08), vidas/XP (Tema 10), matrícula (Tema 02), parámetros (Tema 12), identidad (Tema 01).

**Relaciones:** ItemDefinicion 1→0..\* OfertaCatalogo; OfertaCatalogo 1→0..\* Orden; Orden 1→0..1 ItemInventario; ItemDefinicion 1→0..\* Subasta; Subasta 1→0..\* Puja; Subasta 1→0..1 ItemInventario (adjudicación).

### Preguntas de modelado que quedan abiertas (no tratadas en esta sesión — no asumir respuesta)

- ¿La vida entra al inventario como instancia (`ItemInventario`), o se acredita directo en Tema 10 sin pasar por Mercado? (`diagramas-mercado.md`, §2)
- Resuelto parcialmente por RF-CFG-06: la `Orden` **sí** guarda el precio como snapshot; queda abierto si `OfertaCatalogo` además persiste un precio propio o siempre lee el vigente de Backoffice al momento de mostrarlo.

---

## 6. Máquinas de estado

### Orden (compra directa)
```
CREADA → RESERVA_SOLICITADA → RESERVADA → CONFIRMADA
                ├─→ RECHAZADA_SALDO       (saldo insuficiente en Banco)
RESERVADA ──────┼─→ CANCELADA             (falla la entrega técnica → libera reserva)
                └─→ EXPIRADA              (TTL de la reserva vencido)
```

### Subasta
Versión detallada (la más completa, de la matriz de resiliencia de Subastas — usar esta como autoridad; `EN_CIERRE` en `diagramas-mercado.md` es el mismo concepto que `CLOSING_IN_PROGRESS` aquí, con menos granularidad):

```
DRAFT → SCHEDULED → OPEN (NO_BIDS ↔ ACTIVE_BIDS)
                       ├─→ CANCELLED
                       └─→ CLOSING_IN_PROGRESS
                             ├─ EVALUATING_WINNER
                             ├─ CONFIRMING_LEDGER
                             ├─ CREDITING_INVENTORY
                             ├─ RELEASING_LOSERS
                             ├─ MARKED_DESERTED (sin pujas)
                             └─→ FAILED_SETTLEMENT (AWAITING_MANUAL_OR_CRON_RETRY)
                                     └─→ vuelve a CLOSING_IN_PROGRESS si el reintento tiene éxito
                       → CLOSED
```

El estado intermedio `CLOSING_IN_PROGRESS`/`EN_CIERRE` **no está en el PRD** — se agregó porque el cierre no es instantáneo (confirmar 1 reserva + liberar N); sin él, dos ejecuciones concurrentes del cierre pueden adjudicar dos veces.

### Puja
```
ACTIVA → SUPERADA (liberada, otro alumno pujó más alto)
ACTIVA → GANADORA (al cierre, era la mayor)
ACTIVA → LIBERADA (cierre sin ganar, o cancelación de la subasta)
```

### ItemInventario
```
DISPONIBLE/EQUIPPED → CONSUMIDO (uso único, RF-REC-05)
                    → EXPIRADO (vencimiento, si aplica)
                    → ARCHIVED_READ_ONLY (curso archivado o alumno desmatriculado)
```

---

## 7. Compra directa — flujo end-to-end

### Regla general (decisión #8, confirmada en esta sesión)

El orden de operaciones es **reservar → acreditar/corroborar (vida o ítem) → confirmar el débito recién si la acreditación ya se corroboró**. Esto significa que los rechazos por **regla de negocio** (ej. tope de vidas PAR-12 ya alcanzado) **no necesitan compensación**: nunca se llega a confirmar el débito si la acreditación no puede hacerse. Esto es distinto de una **falla técnica** durante el paso de confirmación (ej. Banco caído, timeout) — ese caso sí necesita liberar la reserva/compensar, porque ahí la falla ocurre después de haber corroborado que la acreditación era válida.

### Secuencia feliz (conceptual, de `diagramas-mercado.md`)

```
Alumno → Gateway → Mercado: POST /mercado/ordenes {ofertaId, idempotencyKey}
Mercado valida: cohorte activa, oferta activa, precio vigente (PAR)
Mercado → Gateway → Banco: POST /banco/reservas {alumno, cohorte, monto, key} → 201 {reservaId}
Mercado crea ItemInventario / corrobora que la acreditación es válida
Mercado → Gateway → Banco: POST /banco/reservas/{id}/confirmar → 200 OK (ledger debitado)
Orden = CONFIRMADA (+ outbox) → 201 al alumno
Mercado publica evento de compra confirmada al bus
Bus notifica a Tema 10 para acreditar vida/equipamiento
```

### Variantes de falla

| Caso | Tratamiento |
|---|---|
| Saldo insuficiente | Banco rechaza en el paso de reserva → orden `RECHAZADA_SALDO`. Nada que compensar. |
| Rechazo por regla de negocio (tope de vidas) | Se corrobora **antes** de confirmar el débito (decisión #8) → no se confirma, no hace falta compensación. |
| Falla técnica en la entrega tras reservar (ej. DB caída) | `liberar(reservaId)` → orden `CANCELADA`. |
| Timeout al confirmar | Reintento con la misma `idempotencyKey`; si persiste, la reserva expira por TTL del Banco y un job de reconciliación cierra la orden. |
| Monedas de otro curso | Rechazo inmediato, sin llegar a reservar (RF-INT-04). |

### Contrato técnico más maduro (fuente: `Comunicacion/Grupo-08-Banco`)

Este documento describe una versión **más concreta y con más detalle técnico** que la de `diagramas-mercado.md` — tratarlo como la referencia técnica principal para diseñar la saga real:

- Cliente → `POST /api/v1/market/orders` `{orderId, itemCode, courseId}` → Mercado responde `202 Accepted {orderId, status:"PROCESSING", streamUrl}`.
- Cliente abre **SSE**: `GET /api/v1/market/orders/stream/{orderId}` (frames `{step, status, message}`).
- Mercado persiste la orden (`market_orders`, estado `PENDING_RESERVATION`) y publica en el tópico `mercado.ordenes` el evento `ORDEN_COMPRA_SOLICITADA` — `{orderId, studentId, courseId, itemCode, amount, currency:"GOLD_COIN", ttlMinutes:5, reason:"ITEM_PURCHASE_DIRECT"}`.
- Banco consume y bloquea saldo, publica en `banco.reservas`: `FONDOS_RESERVADOS {holdId, orderId, studentId, amount, status:"RESERVADO", expiresAt}`, o el camino de rechazo `RESERVA_FONDOS_RECHAZADA`/`FONDOS_INSUFICIENTES` (SSE con código de error `4002`).
- Mercado consume `FONDOS_RESERVADOS`, inserta el ítem en `student_inventory`, emite `ITEM_ACREDITADO` en `mercado.ordenes`.
  - Si falla la persistencia local: Mercado emite `ENTREGA_ITEM_FALLIDA`/`COMPRA_FALLIDA_COMPENSACION`; Banco libera el hold y publica `FONDOS_LIBERADOS`/`RESERVA_LIBERADA`; orden queda `COMPENSATED`.
- Banco publica `DEBITO_FINAL_CONFIRMADO` en `banco.reservas` → Mercado marca la orden `CONFIRMED`, envía el frame SSE final.

Tabla `market_orders`: estados `PENDING_RESERVATION → PROCESSING → CONFIRMED/FAILED/COMPENSATED/REJECTED_INSUFFICIENT_FUNDS`.

> **Nota de nomenclatura:** estos nombres de evento (`ORDEN_COMPRA_SOLICITADA`, `ITEM_ACREDITADO`, etc.) son del contrato **de Banco**, no de Mercado exclusivamente — son parte de un acuerdo bilateral ya escrito. La convención de "MAYÚSCULAS_SNAKE_CASE en inglés" (decisión #4) aplica de acá en adelante a los eventos que **Mercado diseñe y controle unilateralmente** (ver Sección 10); renombrar estos requeriría volver a acordarlo con Banco.

---

## 8. Subastas — diseño detallado

Épica E-07. Contraste con compra directa: la compra directa es una transacción de 3-5 segundos; una subasta es un proceso asíncrono de larga duración (horas/días), con pico de concurrencia estimado en **~120 sesiones simultáneas en el minuto final**.

### 8.1 Opciones de arquitectura evaluadas

| Opción | Mecanismo | Estado |
|---|---|---|
| **1 — Hold Escrow Total** | Banco retiene el 100% de cada puja de cada participante hasta el cierre; al cierre se confirma la ganadora y se liberan todas las demás en batch. | **CONFIRMADA como arquitectura a implementar** (decisión #3) |
| 2 — Leader-Only Floating Hold | Solo la puja líder mantiene un hold activo; se libera en tiempo real al perdedor apenas lo superan. | Analizada y descartada como arquitectura de partida — es superior en UX y en carga sobre Banco, pero el equipo confirmó seguir el contrato que Banco ya documentó preliminarmente (`HOLD_INCREASE_REQUESTED`, hold por puja), que es compatible con la Opción 1, no con esta. |
| 3 — Collateral Margin & Settlement | Colateral fijo (ej. 20% del valor) + liquidación de la diferencia al cierre; default del ganador → confisca colateral y pasa al segundo postor. | Descartada: solvencia parcial/débil, riesgo de impago real en contexto educativo, complejidad de cascada muy alta. |

**Por qué queda documentado el análisis completo pese a no elegir la Opción 2:** dejar registro de que la Opción 2 es objetivamente mejor en UX y carga en Banco ayuda a entender por qué el diseño de Opción 1 necesita mecanismos de mitigación extra (ver 8.2, especialmente el batch release) — no es una limitación arbitraria, es una decisión consciente de compatibilidad con lo que Banco ya construyó.

### 8.2 Los 5 errores críticos y sus soluciones

1. **Desincronización de expiración (TTL Drift / Anti-sniping):** el `HoldExpirationScheduler` de Banco puede liberar el hold antes de tiempo si Mercado implementa prórrogas (soft close) sin avisarle. → **Grace Period**: `ttlSeconds` = duración de la subasta + 30 min de buffer; el cierre lo decide Mercado explícitamente, nunca el timer de Banco. Contrato de extensión: `HOLD_EXTEND_REQUESTED` o re-emitir `HOLD_INCREASE_REQUESTED`.
2. **Doble cierre (Double-Hammer Race Condition):** dos réplicas de Mercado disparando el cierre a la vez → doble adjudicación. → Bloqueo optimista obligatorio (`@Version` en `MarketAuction`, transición `OPEN → CLOSING_IN_PROGRESS`) + coordinación distribuida (ShedLock sobre Postgres, o partición por `auctionId` en Kafka).
3. **Vidas subastadas rompen el tope de Tema 10:** si el ganador ya está en el máximo de vidas al cerrar, Tema 10 rechaza la acreditación. → **Las vidas no son subastables por definición de producto** — solo cosméticos, títulos, escudos, multiplicadores sin tope rígido.
4. **Tormenta de liberaciones al cierre (Thundering Herd Release):** con la Opción 1 confirmada, este riesgo es **más relevante, no menos** — hay que liberar N holds perdedores de una sola vez. → **Batch Release**: evento consolidado `SUBASTA_CERRADA`/`AUCTION_CLOSED` con lista de `holdIds`, o comando `HOLD_RELEASE_BATCH_REQUESTED(auctionId)` en Banco. Nunca un `for` síncrono llamando de a uno.
5. **Desconexión móvil inmediata tras ofertar:** el alumno puede perder señal antes de recibir el `202 Accepted`. → `X-Idempotency-Key` único por intento + reconexión SSE con `Last-Event-ID`.

### 8.3 Matriz de fallos por microservicio

| Microservicio | Momento | Impacto | Solución |
|---|---|---|---|
| Banco | Puja inicial (`HOLD_CREATE`) | Reserva no se asienta | Rechazo limpio: puja → `REJECTED_BANK_UNAVAILABLE`, sin impacto contable |
| Banco | Cierre (`HOLD_CONFIRM`) | Débito definitivo no registrado | Estado `CLOSING_PENDING_SETTLEMENT` + Transactional Outbox Worker con backoff exponencial |
| Banco | Liberación (`HOLD_RELEASE`) | Perdedores no desbloqueados | Tabla `market_pending_refunds` + cronjob de re-emisión; alerta en Backoffice si un hold lleva &gt;15 min sin liberar |
| Mercado | Crash del pod durante cierre | Ganador elegido pero comandos no enviados | `AuctionRecoveryService` al arrancar: busca subastas vencidas en `OPEN`/`CLOSING_IN_PROGRESS` y reanuda idempotentemente (`orderId = auctionId`) |
| Kafka | Caída de clúster/partición | Comandos no entregados | **Transactional Outbox Pattern**: tabla `market_outbox` en la misma transacción ACID |
| Inventario | Fallo de persistencia al acreditar | Pagó en Banco pero el ítem no se creó | **Saga Reversal**: `COMPENSATION_REFUND_REQUESTED`, subasta marcada `FAILED_COMPENSATED` |

### 8.4 Doctrina de idempotencia en 3 capas

1. **Clave natural**: `commandId = SHA-256(auctionId + studentId + bidSequenceNumber)`.
2. **Tabla de deduplicación en Banco** (`processed_commands`): `command_id PRIMARY KEY`, `producer_service`, `command_type`, `processed_at`, `response_payload JSONB`.
3. **Bloqueo optimista en Mercado**: `@Version` sobre `MarketAuction`.

**Envoltura estándar obligatoria de todo evento** (5 campos, estándar de plataforma): `{eventId, eventType, timestamp, producer, payload}`.

### 8.5 Contratos de eventos de la saga de subasta

Estos son un contrato **bilateral con Banco** — los nombres `HOLD_*` ya están en inglés y ya son parte de lo que Banco espera recibir; no se renombran.

| Evento | Dirección | Tópico | Payload clave |
|---|---|---|---|
| `HOLD_CREATE_REQUESTED` | Mercado → Banco | `bank.holds.commands` | `commandId, auctionId, studentId, amount, orderType:"AUCTION_BID", ttlSeconds, graceBufferSeconds` |
| `HOLD_CREATED` | Banco → Mercado | `bank.holds.events` | `commandId, holdId, auctionId, studentId, amount, status:"PENDING", expiresAt` |
| `HOLD_INCREASE_REQUESTED` | Mercado → Banco | `bank.holds.commands` | `commandId, holdId, auctionId, currentAmount, newTotalAmount, incrementalAmount` |
| `HOLD_INCREASED` | Banco → Mercado | `bank.holds.events` | `commandId, holdId, auctionId, newTotalAmount, status:"PENDING"` |
| `HOLD_CONFIRM_REQUESTED` | Mercado → Banco | `bank.holds.commands` | `commandId, holdId, auctionId, winnerStudentId, confirmedAmount` |
| `HOLD_CONFIRMED` | Banco → Mercado | `bank.holds.events` | `commandId, holdId, auctionId, debitedAmount, ledgerEntryId, status:"COMMITTED"` |
| `HOLD_RELEASE_REQUESTED` | Mercado → Banco | `bank.holds.commands` | `commandId, holdId, auctionId, studentId, releaseReason` |
| `HOLD_RELEASED` | Banco → Mercado | `bank.holds.events` | `commandId, holdId, auctionId, releasedAmount, status:"RELEASED", releaseReason` |

Estos dos sí son propios de Mercado y **se renombran** según la convención confirmada (decisión #4):

| Nombre original (fuente: `03-contratos-eventos-e-idempotencia.md`) | Nombre confirmado | Tópico | Consumidores |
|---|---|---|---|
| `SUBASTA_ADJUDICADA` | **`AUCTION_AWARDED`** | `mercado.subastas` | Notificaciones (11), Backoffice (12), Cursos (02) |
| `OFERTA_SUPERADA` | **`BID_OUTBID`** | canal SSE / `notificaciones.alertas` | Notificaciones (11), Web & Móvil |

Particionamiento Kafka: `partitionKey = auctionId` en todos los tópicos de subasta, para garantizar orden FIFO por subasta.

### 8.6 Directivas de implementación (resumen ejecutivo)

1. Nunca emitir a Kafka fuera de una transacción de BD local — tabla `outbox_events` en Mercado y en Banco.
2. TTL con gracia de +30 min; el martillo del cierre es exclusivo de Mercado, nunca el scheduler de Banco.
3. No permitir subastar consumibles con tope — solo cosméticos/equipables sin límite rígido.
4. Alerta temprana: métrica `auctions_closing_pending_seconds`, crítica si &gt; 120 segundos en `CLOSING_IN_PROGRESS`.

---

## 9. Contratos de integración por dependencia

### 9.1 Banco (Tema 08) — dependencia crítica bloqueante

Protocolo obligatorio (saga de 3 pasos, no es "descuento y después entrego"):
```
1. reservar(alumno, curso, monto, idempotencyKey) → reservaId | RECHAZADA(saldo insuficiente)
2. (mercado entrega el ítem / corrobora la acreditación)
3. confirmar(reservaId) → descuento efectivo en el ledger
   o liberar(reservaId) → devolución sin costo
```
Doctrina: sincrónico vía Gateway para comandos con respuesta inmediata, asincrónico vía Kafka para hechos consumados. Envoltura estándar de 5 campos en todo mensaje. Broker: Kafka (`kafka:9092`), semántica *at-least-once*, `partitionKey = studentId` para orden FIFO por alumno en compra directa, `partitionKey = auctionId` en subastas.

Puntos que había que acordar y quedaron resueltos/aclarados en esta sesión: modelo de hold en subastas = Opción 1 (decisión #3); validación de reglas de negocio antes de confirmar débito, sin necesidad de compensación para ese caso (decisión #8).

Puntos que **siguen sin acordar formalmente**: expiración de reserva en compra directa (TTL + job de reconciliación — "acordar, no asumir", según el propio kickoff); códigos de error diferenciados completos.

### 9.2 Cursos y Matrícula (Tema 02)

- **Consulta síncrona** (Mercado → Cursos vía Gateway): `GET /api/v1/courses/{courseId}/students/{studentId}/enrollment-status` → `{courseId, studentId, enrolled, status, enrolledAt}`. Si `enrolled:false` → Mercado responde `403 {NOT_ENROLLED_IN_COURSE}`.
- **Tópico Kafka `cursos.ciclo-vida`**, consumido en paralelo por Mercado (`mercado-cursos-group`), Banco y Notificaciones:
  - `CURSO_ARCHIVADO` (partitionKey `courseId`): Mercado transiciona `student_inventory` de ese curso a `ARCHIVED_READ_ONLY`, bloquea nuevas órdenes/compras/subastas.
  - `ALUMNO_DESMATRICULADO` (partitionKey `studentId`): Mercado congela los ítems del alumno en esa cohorte, desequipa automáticamente cualquier ítem activo.
- **Decisión #7 (confirmada):** Cursos debe **bloquear el archivado** de un curso mientras tenga subastas activas, en vez de cancelarlas automáticamente. **Pendiente de comunicación formal con el equipo de Cursos** — es una restricción que ellos deben implementar de su lado.
- Esquema relevante: `student_inventory(inventory_item_id PK, student_id, course_id, item_code, state, charges, UNIQUE(student_id, course_id, inventory_item_id))`.

### 9.3 Identidad y Gateway (Tema 01)

- Cabeceras inyectadas a Mercado: `X-User-Id`, `X-Roles` (CSV), `X-User-Email`, `X-Forwarded-For`. Mercado nunca valida JWT ni recibe credenciales.
- Roles (RBAC local en Mercado, "validar no es autorizar"):
  - `ROLE_STUDENT`: consulta catálogo, compra, puja, gestiona mochila.
  - `ROLE_PROFESSOR`: curaduría de ofertas por cohorte (equipamiento — decisión #5), lanza/cancela subastas.
  - `ROLE_ADMIN`: auditoría global y parametrización.
- Errores: `401 {INVALID_OR_EXPIRED_TOKEN}` (en Gateway), `403 {ROLE_NOT_PERMITTED}` (en Mercado).
- Sincronización opcional: tópico `users.lifecycle.events`, evento `USER_PROFILE_UPDATED` → tabla de lectura rápida `cached_users` para mostrar nombres en subastas sin llamar por HTTP.

### 9.4 Motor de Desafíos (Tema 03) + Runner (Tema 05)

- **Consulta síncrona** (Desafíos → Mercado vía Gateway): `GET /api/v1/market/inventory/students/{studentId}/active-items?courseId={courseId}` — devuelve el "Executable Directive Pattern": lista de ítems equipados con `verb` (`ABSORB_FAILURE`, `XP_MULTIPLIER`, `COIN_MULTIPLIER`) y `actionParams`. Timeout de 800ms → si Mercado no responde, Desafíos asume "sin ítems equipados" y no bloquea al alumno.
- **Tópico `desafios.resultados`**, evento `DESAFIO_RESUELTO`: Mercado (`mercado-consumo-group`) descuenta cargas de `student_inventory`; si llegan a 0, transiciona a `CONSUMED`. Idempotencia por `attemptId`.
- Mercado emite su propio evento de auditoría `ITEM_CONSUMIDO` en el tópico `mercado.inventario`.
- **Este es el mecanismo real y concreto que existe hoy** para el consumo de equipamiento durante un desafío — ver la discusión completa y su estado (pendiente #11) en la Sección 12-B, porque el equipo tiene propuestas nuevas en camino que podrían reemplazarlo.
- Catálogo canónico de 16 consumibles (familias SHIELD, XP, COIN, LIFE — con tiers, precios y efectos) documentado íntegramente en este contrato.

### 9.5 Backoffice (Tema 12)

- **Asíncrono**: tópico `backoffice.parametros`, evento `PARAMETRO_ACTUALIZADO` → Mercado invalida caché local y aplica **hot-reload sin downtime**. Carga inicial en `@PostConstruct` con fallback a `application.yml`.
- **Sincrónico**: `GET /api/v1/market/admin/metrics?courseId=` (rol `PROFESSOR`), snapshot agregado con frescura máxima de 15 minutos.
- **Pendiente #9 (no resuelto):** si además hace falta poder pedir PAR-06/07 por HTTP vía Gateway en el momento, en vez de depender solo del evento — el equipo dice que *puede* hacer falta pero todavía no está definido.

### 9.6 Notificaciones (Tema 11)

Dueño del contrato de eventos de toda la plataforma. Consume los eventos de Mercado (ver catálogo completo en Sección 10) para notificar: nuevo ítem en catálogo, subasta que arranca, puja superada, subasta ganada/perdida/cancelada.

### 9.7 Ranking / Roadmap y Progreso (Tema 10)

- Acreditación de vidas y aplicación del efecto de equipamiento — **mecanismo exacto pendiente** (#11).
- Consume `DESAFIO_RESUELTO` en paralelo a Mercado, para XP y leaderboard — no participa en la decisión de consumir equipamiento (ver 9.4).
- La validación de tope de vidas (PAR-12) al comprar se resuelve con la decisión #8: reservar → corroborar la acreditación con Tema 10 → confirmar el débito solo si Tema 10 la aceptó.

---

## 10. Catálogo de eventos que Mercado publica/consume

Convención confirmada (decisión #4): **MAYÚSCULAS_SNAKE_CASE, en inglés**, para todo evento que Mercado diseñe y controle. La siguiente tabla trae el nombre tal como aparece en cada documento fuente junto con el nombre a usar de acá en adelante.

| Nombre en la fuente | Fuente | Nombre confirmado a usar |
|---|---|---|
| `mercado.oferta.publicada` | kickoff §5.4 | `CATALOG_OFFER_PUBLISHED` |
| `mercado.compra.confirmada` | kickoff §5.4 | `PURCHASE_CONFIRMED` |
| `mercado.item.consumido` / `ITEM_CONSUMIDO` | kickoff / Comunicacion Grupo-03 | `ITEM_CONSUMED` |
| `mercado.subasta.abierta` | kickoff §5.4 | `AUCTION_OPENED` |
| `mercado.subasta.cerrada` | kickoff §5.4 | `AUCTION_CLOSED` |
| `mercado.subasta.cancelada` | kickoff §5.4 | `AUCTION_CANCELLED` |
| `mercado.puja.superada` / `OFERTA_SUPERADA` | kickoff / Subastas 03 | `BID_OUTBID` |
| `SUBASTA_ADJUDICADA` | Subastas 03 | `AUCTION_AWARDED` |

Eventos que Mercado **consume** (de otros dueños, no se renombran): `CURSO_ARCHIVADO`, `ALUMNO_DESMATRICULADO` (Tema 02), `DESAFIO_RESUELTO` (Tema 03), `PARAMETRO_ACTUALIZADO` (Tema 12), `HOLD_CREATED`/`HOLD_INCREASED`/`HOLD_CONFIRMED`/`HOLD_RELEASED` (Tema 08), `FONDOS_RESERVADOS`/`RESERVA_FONDOS_RECHAZADA`/`FONDOS_INSUFICIENTES`/`FONDOS_LIBERADOS`/`RESERVA_LIBERADA`/`DEBITO_FINAL_CONFIRMADO` (Tema 08, saga de compra directa).

Regla de diseño de eventos (del kickoff, sigue vigente): **un evento es un hecho consumado**, nombre en pasado, sin pedir respuesta. Si hace falta una respuesta para continuar, es sincrónico por el Gateway, no un evento.

---

## 11. Backlog propuesto para Sprint 1 (Sprint 0 del equipo)

> ⚠️ **El alcance real de Sprint 1 todavía NO está definido** (decisión pendiente #12, ver Sección 12-B). Lo que sigue es la **propuesta inicial del equipo**, presentada en su Sprint 0, **aún no ratificada** por el PO. La definición real se cierra en las próximas horas — tratar esta sección como punto de partida, no como compromiso cerrado.

### Definition of Done propuesta (11 puntos, resumen)

Criterios de aceptación con escenarios de falla (no solo camino feliz, formato Gherkin) · code review aprobado · **toda comunicación entre microservicios pasa por el Gateway** (verificado en review) · borrado lógico respetado · **al menos un escenario de concurrencia probado** si la historia toca saldo o inventario · formato de eventos conforme al contrato acordado · probado manualmente por alguien distinto de quien codeó · sin errores/warnings en consola durante demo · endpoints nuevos documentados antes de cerrar · tarjeta de Taiga reflejando estado real · aprobación del PO antes de "Done".

### Capacidad efectiva del equipo

| Indicador | Valor |
|---|---|
| Duración del Sprint | 10 días hábiles |
| Integrantes | 11 |
| Capacidad teórica | 880 h |
| Ceremonias Scrum | 71,5 h |
| **Capacidad efectiva** | **323,4 h (~37% de un sprint normal)** |

### Épica M-00 — Spike técnico: contrato de reserva/confirmación con el Banco

Preguntas del spike (estado: **resuelto parcialmente**):
- ¿El endpoint de reserva bloquea con TTL, o Mercado confirma/libera explícitamente? — Resuelto conceptualmente: reservar→confirmar/liberar (Sección 9.1).
- ¿Cómo se identifica una operación para evitar dobles cobros? — Resuelto: `idempotencyKey`.
- ¿Qué pasa si dos compras simultáneas agotan el mismo saldo? — Lo arbitra Banco.
- ¿Sirve el mismo contrato para subastas? — **Aclarado en esta sesión**: la mención de "Fase 2" era un error; subastas es Fase 3 y usa un contrato de holds más elaborado (Sección 8), no el mismo de compra directa simple.
- Stock/límite de compras de equipamiento — **Resuelto en esta sesión (decisión #6): no hay stock, disponibilidad ilimitada.** Este ítem, que el propio Sprint 0 marcaba como abierto, ya no lo está.
- Quedan abiertos, sin tratar en esta sesión: canal de notificación del resultado, parámetros exactos de reintentos/timeout.

### Épica M-01 — Catálogo de ítems

- **HU-01.1**: Como alumno, ver la lista de ítems disponibles en mi curso (nombre, tipo, precio). Solo ítems activos de mi curso-cohorte con precio vigente.
- **HU-01.2**: Como profesor, el catálogo muestra solo ítems del curso-cohorte correcto.
- Tareas: modelar `ItemDefinicion` (tipo, precio, curso-cohorte, activo/inactivo); endpoint de listado filtrado por curso-cohorte del token; leer precios por defecto de PAR-06/PAR-07; UI de catálogo (alcance escritorio).

### Épica M-02 — Compra directa con reserva contra el Banco

- **HU-02.1** — Confirmar compra y ver resultado:
  - Escenario 1 (éxito): saldo suficiente + ítem disponible → reserva → confirmación → ítem acreditado, monedas descontadas.
  - Escenario 2 (falla técnica en confirmación): reserva se libera, ítem no se acredita, alumno recupera saldo. *(Nota: este es el caso de falla técnica genuina durante el paso de confirmación — distinto del rechazo por regla de negocio de la decisión #8, que se resuelve antes de llegar a confirmar.)*
  - Escenario 3 (monedas de otro curso): rechazo, se informa que las monedas no pertenecen a ese curso.
- **HU-02.2**: feedback visual durante la compra (botón deshabilitado + spinner) para evitar doble clic.
- Tareas: flujo reserva→confirmación sincrónico contra Banco; validar en orden disponibilidad→pertenencia al curso→saldo→moneda del curso correcto; reintentos automáticos con compensación al agotarlos (caso de falla técnica); pruebas de concurrencia sobre compras simultáneas del mismo ítem.

---

## 12. Estado de decisiones — confirmadas vs. pendientes reales

### 12-A. Decisiones confirmadas en esta sesión

Estas se toman como definitivas para el diseño de Sprint 1, aunque los documentos fuente originales todavía no las reflejen:

1. **Fase de subastas = Fase 3.** La mención de "Fase 2" en el spike M-00 del Sprint 0 es un error de redacción del equipo.
2. **Broker de mensajería = Kafka**, definitivo para toda la plataforma — no solo para Mercado/Subastas.
3. **Modelo de hold en subastas = Opción 1 (Hold Escrow Total)**, confirmada como arquitectura a implementar, no como paso intermedio — se respeta el contrato que Banco ya documentó preliminarmente (`HOLD_INCREASE_REQUESTED`) como dato de entrada fijo.
4. **Nomenclatura de eventos propios de Mercado: MAYÚSCULAS_SNAKE_CASE, en inglés**, sin excepción (ver tabla completa en Sección 10).
5. **Origen del equipamiento con efecto mecánico:** lo define el profesor, por curso-cohorte (curaduría propia, no catálogo global fijo).
6. **Sin límite de stock:** la oferta de catálogo tiene disponibilidad ilimitada mientras esté activa.
7. **Archivado de curso con subasta abierta:** Cursos (Tema 02) debe bloquear el archivado hasta que no queden subastas activas — pendiente de comunicación formal con ese equipo para que lo implementen.
8. **Validación de reglas de negocio sin compensación:** reservar → corroborar la acreditación (vida o ítem) → confirmar el débito solo si la acreditación se corroboró. Aplica igual a vidas y a ítems.

### 12-B. Pendientes genuinos — sin resolver, no asumir respuesta

9. **Sincronización de PAR-06/PAR-07 vía Gateway:** hoy la base es el evento `PARAMETRO_ACTUALIZADO` con caché local; puede hacer falta además un endpoint sincrónico, pero no está definido.
10. **Desmatriculación a mitad de curso:** hay una propuesta razonable (mismo criterio que curso archivado — solo lectura), pero **no validada por el equipo todavía**.
11. **Consumo de equipamiento con efecto mecánico — quién invoca a quién:** existen dos piezas documentadas hoy (la recomendación especulativa del kickoff, y el contrato concreto ya escrito con Motor de Desafíos vía consulta síncrona + reporte asíncrono), pero **ninguna está confirmada como definitiva** — hay propuestas nuevas del equipo en camino que todavía no están en ningún archivo.
12. **El alcance real de Sprint 1 en sí.** La propuesta del Sprint 0 (M-00 + M-01 + M-02) es un punto de partida, no un acuerdo ratificado. Se define en las próximas horas.

### 12-C. Otras preguntas menores abiertas en las fuentes (no tratadas en esta sesión)

Existen en los documentos originales, nadie las validó en esta conversación — no asumir ninguna respuesta:

- ¿La vida entra al inventario como instancia local, o se acredita directo en Tema 10? (`diagramas-mercado.md` §2)
- Canal de notificación del resultado de una compra y parámetros exactos de reintentos/timeout con Banco (Sprint 0, M-00).
- Vencimiento de ítems: ¿por fecha, por cierre de curso, o ambos? No está en el PRD (kickoff §4, pregunta 7).
- Intercambio entre alumnos ("podría ser" en la propuesta de arquitectura): ¿entra en algún sprint? Si entra, cuidado con RF-INT-01 (nunca se generan monedas por intercambio).
- Gap de riesgos: el PRD no registra riesgos específicos de la economía de intercambio (Sección 4.5 de este documento) — vale la pena que el equipo lo agregue a su propio registro.

---

## 13. Referencias

| Documento | Contenido |
|---|---|
| `README.md` (raíz) | Índice general del repo y su estructura de carpetas |
| `mercado-kickoff (1).md` | Documento de arranque completo: alcance, dependencias, preguntas al PO, modelado, arquitectura back/front end, Scrum/Taiga, reparto del equipo |
| `diagramas-mercado.md` | Borradores Mermaid: contexto C4, modelo de dominio, máquinas de estado, secuencias de compra directa y cierre de subasta, ciclo de vida de cohorte |
| `Mercado/Subastas/README.md` | Índice del paquete documental de Subastas (incluye HTML interactivos no cubiertos en detalle aquí) |
| `Mercado/Subastas/01-analisis-opciones-arquitectura.md` | Las 3 opciones de arquitectura de holds para subastas, con matriz comparativa y dictamen |
| `Mercado/Subastas/02-matriz-fallos-resiliencia-y-soluciones.md` | Los 5 errores críticos, matriz de fallos por microservicio, máquina de estados resiliente |
| `Mercado/Subastas/03-contratos-eventos-e-idempotencia.md` | Canales de comunicación, doctrina de idempotencia en 3 capas, los 10 contratos de eventos con JSON completo |
| `PRD-Plataforma-Gamificada-TP.pdf` | Fuente de verdad oficial del producto: fasificación, roles, todos los RF-*/PAR-*, riesgos, glosario |
| `Sprint0_Propuesta_Mercado.pdf` | DoD del equipo, cálculo de capacidad, épicas M-00/M-01/M-02 con historias de usuario Gherkin |
| `Comunicacion/Grupo-01-Identidad-y-Gateway/flujo-comunicacion-usuarios.md` | Cabeceras de confianza, roles RBAC, errores de auth, evento de sincronización de perfil |
| `Comunicacion/Grupo-02-Cursos-y-Matricula/flujo-comunicacion-cursos.md` | Endpoint de pertenencia, eventos de ciclo de vida del curso |
| `Comunicacion/Grupo-03-Motor-de-Desafios/flujo-comunicacion-motor-desafio.md` | Consulta de ítems equipados, evento `DESAFIO_RESUELTO`, catálogo completo de 16 consumibles |
| `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md` | Saga completa de compra directa con SSE, contratos de eventos, tabla `market_orders` |
| `Comunicacion/Grupo-12-Backoffice/flujo-comunicacion-backoffice.md` | Parámetros económicos, hot-reload, endpoint de métricas administrativas |
| `Workflow/README.md` | Convenciones de branching y commits del repo completo |
