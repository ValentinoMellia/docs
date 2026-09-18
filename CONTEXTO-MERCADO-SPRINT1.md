# Contexto consolidado — Mercado (Tema 09 / Grupo 11) para Sprint 1

> **Estado: cerrado para arrancar Sprint 1.** Este documento incorpora el contexto disponible al 16/09/2026 **más los 6 pendientes genuinos que el equipo cerró en vivo el 18/09/2026**, más una auditoría línea por línea de las fuentes originales (`diagramas-mercado.md`, `Mercado/Catalogos/README.md`, `Mercado/Subastas/*`, `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md`, `README.md` raíz) que encontró contradicciones concretas con las decisiones ya tomadas. Ya no quedan pendientes genuinos bloqueantes para Sprint 1 (ver Sección 12).
>
> **Importante — las fuentes originales todavía tienen el contenido viejo.** La auditoría de la Sección 12-D detalla exactamente qué está desactualizado en cada archivo fuente (stock finito, catálogo fijo de plantillas, Opción 2 de subastas como objetivo, eventos en español, orden de liquidación invertido, tres enumeraciones distintas de estado de subasta). Mientras esos archivos no se corrijan físicamente, **este documento manda** sobre todos esos puntos — no la fuente original.

> **Alcance de este documento:** catálogo, compra directa, subastas. **El inventario/mochila del alumno ya NO es de Mercado** (pendiente #13 resuelto: pasa a Grupo 12/Banco). No es un resumen: es un análisis exhaustivo pensado para que el agente que arranca el ciclo SDD (`sdd-explore` → `sdd-propose` → `sdd-spec` → `sdd-design` → `sdd-tasks` → `sdd-apply`) tenga todo el contexto técnico y de negocio sin tener que reconstruirlo leyendo 10 documentos dispersos.

---

## 1. Propósito y cómo leer este documento

Este archivo es un **mapa de navegación con profundidad técnica real**, no un índice ni un resumen ejecutivo. Cita nombres de eventos, campos de payloads, IDs de requerimiento (`RF-XXX`, `PAR-XXX`) y fragmentos de contratos tal como quedaron redefinidos, para que las decisiones de diseño del Sprint 1 se puedan tomar sin adivinar nada.

**Qué NO es:** no reemplaza a los documentos fuente para todo lo que no está listado como decisión de este documento (ver [Sección 13 — Referencias](#13-referencias)). Cuando haga falta el detalle extremo que no cambió — el diagrama Mermaid completo, el JSON exacto de un contrato no tocado — hay que ir a la fuente original, sabiendo que sus partes de dominio, stock y subastas están desactualizadas (Sección 12-D).

### TL;DR de las decisiones ya tomadas (detalle completo en Sección 12)

**Del 16/09 (8 decisiones):**
1. Subastas = **Fase 3** (confirmado; "Fase 2" en el spike M-00 fue error de redacción).
2. Broker de mensajería = **Kafka**, definitivo para toda la plataforma.
3. Modelo de holds en subastas = **Opción 1 (Hold Escrow Total)**, confirmada como arquitectura **definitiva**, no como paso intermedio hacia la Opción 2.
4. Nomenclatura de eventos propios de Mercado = **MAYÚSCULAS_SNAKE_CASE, en inglés**, sin excepción.
5. El equipamiento con efecto mecánico lo define **el profesor, por curso-cohorte** (curaduría, no catálogo global fijo de ítems concretos).
6. **Sin límite de stock** en la oferta de catálogo.
7. Cursos (Tema 02) debe **bloquear el archivado** de un curso mientras tenga subastas activas.
8. Validaciones de negocio se resuelven **reservando → corroborando la acreditación → confirmando el débito recién si la acreditación se corroboró**.

**Del 18/09 (6 pendientes cerrados en esta sesión):**
9. **Sincronización de PAR-06/07: se agrega endpoint sincrónico** además del evento `PARAMETRO_ACTUALIZADO` + caché local.
10. **Desmatriculación a mitad de curso: mismo criterio que curso archivado** (solo lectura, sin nuevas acciones).
11. **Consumo de equipamiento con efecto mecánico: sale completo del alcance de Mercado.** Pasa a ser un contrato exclusivo entre Motor de Desafíos (Tema 03) y Grupo 12/Banco.
12. **Alcance de Sprint 1: se ratifica M-00 + M-01 + M-02**, ajustadas al recorte de inventario y al catálogo por plantillas.
13. **Inventario/mochila del alumno: sale de Mercado, pasa a Grupo 12 (Banco).**
14. **Catálogo por plantillas: se adopta.** Ya no hay tiers fijos definidos por Mercado; el profesor configura parámetros sobre un tipo de plantilla.

Ver Sección 12-D para la lista de contradicciones que la auditoría encontró en las fuentes originales (independientes de estos 14 puntos, pero igual de bloqueantes para diseñar sin ambigüedad).

---

## 2. El sistema completo (Aula Quest) y el lugar de Mercado en él

**Aula Quest** es una plataforma de e-learning gamificada — Trabajo Práctico Integrador de Programación IV Back End, TUP UTN FRC. Se construye como un ecosistema de microservicios repartido en **12 temas/grupos**, cada uno dueño exclusivo de su base de datos y de un dominio.

Cada curso es un roadmap de aprendizaje incremental. El alumno resuelve desafíos, acumula XP, monedas, insignias, vidas y equipamiento, compite en un ranking por curso, y es asistido (nunca reemplazado) por agentes de IA con restricciones pedagógicas estrictas.

### Reglas de plataforma no negociables (aplican a todos los temas, incluido Mercado)

1. El **API Gateway (Tema 01)** es la única puerta de entrada. Nada de comunicación directa entre microservicios.
2. Cada servicio es **dueño exclusivo de su base**. Nadie lee la tabla del vecino.
3. Todo lo sincrónico sale y entra por el Gateway; todo lo asincrónico va por el **bus de eventos Kafka** (decisión #2).
4. **No hay borrado físico** de nada (salvo el chat social) — todo es baja lógica (RF-NFR-01).
5. Cada entidad tiene **un dueño único**.

### Temas relevantes para Mercado

| Tema | Nombre | Relación con Mercado |
|---|---|---|
| **01** | Identidad y Gateway | Perimetral: valida JWT, inyecta `X-User-Id`/`X-Roles`/`X-User-Email`. Mercado nunca ve credenciales, solo cabeceras de confianza. |
| **02** | Cursos y Matrícula | Dueño del curso-cohorte y su ciclo de vida (draft→activo→archivado). Publica eventos de archivado/desmatriculación que Mercado debe procesar (decisión #10: desmatriculación = mismo tratamiento que archivado). |
| **03** | Motor de Desafíos | Consulta directamente a **Grupo 12/Banco** (ya no a Mercado, decisión #11) qué ítems tiene equipados el alumno antes de correr un desafío. |
| **05** | Runner | Sandbox Docker efímero que ejecuta tests para el Motor de Desafíos (sin relación directa con Mercado). |
| **08 / Grupo 12** | Banco | **Dependencia crítica bloqueante.** Dueño exclusivo del ledger y los saldos. **Desde la decisión #13, también dueño del inventario/mochila del alumno** (`ItemInventario`, su ciclo de vida, y el consumo de equipamiento durante un desafío). Mercado nunca guarda saldo ni inventario — todo pasa por reserva/confirmación/liberación y por acreditación de ítems contra Banco. |
| **09** | **Mercado** | Este tema: catálogo (por plantillas configurables), compra directa, subastas. **Ya no incluye inventario/mochila del alumno.** |
| **10** | Ranking / Roadmap y Progreso | **Dependencia crítica.** Dueño de XP, niveles, vidas, logros e insignias. Mercado vende vidas y equipamiento; Tema 10 gobierna sus reglas (ej. tope de vidas PAR-12) y Banco/Grupo 12 acredita el efecto en inventario. |
| **11** | Notificaciones (Social) | **Dueño del contrato de eventos de toda la plataforma.** Consume los eventos de Mercado para notificar al alumno. |
| **12** | Backoffice | Administra los parámetros económicos globales (PAR-01 a PAR-24). Mercado consume PAR-06 y PAR-07 (precios) por evento **y ahora también por endpoint sincrónico** (decisión #9). |

> **Nota de numeración:** el equipo usa "Grupo 12" para referirse al equipo de Banco en Taiga — no confundir con "Tema 12" (Backoffice) en la numeración de temas del PRD. Son dos cosas distintas: Backoffice sigue siendo Tema 12 para parámetros económicos; Banco (a quien se le cede inventario) es el Grupo 12 del equipo, independiente de esa numeración de temas.

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
| **Pedido para empezar** | Catálogo · Compra contra reserva del Banco · ~~Inventario del alumno por curso~~ · ~~Consumo de ítems~~ (ambos tachados: pasaron a Grupo 12, decisión #13) |
| **Para más adelante** | Subastas con ventana temporal · Acceso móvil a subastas · Vencimiento de ítems |
| **Podría ser** | Intercambio entre alumnos · Ítems por temporada · Catálogo configurable por curso (**ya no es "podría ser": decisión #14, confirmado**) |

### 3.3 Confirmado

- **Subastas = Fase 3**, sin ambigüedad (decisión #1).
- El TPI avanza sobre alcance de Fase 2 (compra directa, catálogo) en sprints tempranos — decisión de secuenciación del trabajo práctico, no contradicción con el PRD.

### 3.4 Alcance recortado — inventario (RESUELTO, decisión #13)

**Confirmado el 18/09/2026: el inventario/mochila del alumno sale de Mercado y pasa a Grupo 12 (Banco).** El alcance de trabajo de Mercado queda acotado a:

- **Catálogo** — definición de la oferta disponible por curso-cohorte, **por plantillas configurables** (decisión #14).
- **Compra directa** — reserva/confirmación contra Banco, y solicitud de acreditación del ítem contra Grupo 12 (ya no inserción local en `ItemInventario`).
- **Subastas** — todo lo de la Sección 8, con la misma lógica: el ítem ganado se acredita contra Grupo 12, no en una tabla propia de Mercado.

Queda **fuera** de Mercado, de forma definitiva:
- El **inventario/mochila del alumno** y su ciclo de vida completo.
- El **consumo de ítems** durante un desafío y sus **efectos mecánicos** (decisión #11: contrato exclusivo Motor de Desafíos ↔ Grupo 12).
- El evento `ITEM_CONSUMED` (ya no lo publica Mercado).

**Impacto en el resto del documento:** el modelo de dominio (Sección 5) ya no incluye `ItemInventario`; las máquinas de estado (Sección 6) ya no incluyen la de inventario; el flujo de compra (Sección 7) y de subastas (Sección 8) sustituyen "crear `ItemInventario`" por "solicitar acreditación a Grupo 12"; los contratos de integración (Sección 9) reflejan a Grupo 12 como dueño de inventario.

---

## 4. Reglas de negocio cerradas (fuente: PRD — "no sujetas a replanteo" salvo consenso con el PO)

### 4.1 Sistema de intercambio (Sección 10 del PRD — el núcleo de Mercado)

- **RF-INT-01**: las monedas solo se canjean por **vidas o equipamiento**; nunca al revés; **nunca se obtienen monedas por intercambio**.
- **RF-INT-02**: solo las monedas son intercambiables.
- **RF-INT-03**: dos modalidades — **compra directa** (precio fijo) y **subasta** (mayor oferta se lleva el ítem).
- **RF-INT-04**: las monedas usadas deben pertenecer **al mismo curso** donde se realiza el intercambio.
- **RF-INT-05** — reglas completas de subasta:
  - el PROFESOR lanza el asset y define **duración** y (opcionalmente) **puja mínima** de entrada;
  - al pujar, las monedas quedan **bloqueadas/reservadas** hasta el cierre;
  - el pujador puede **aumentar** su oferta, **nunca retirarla ni bajarla**;
  - al cierre: al ganador se le descuentan las monedas y recibe el asset (acreditado en Grupo 12); a los demás se les **liberan las reservas sin costo**;
  - subasta sin pujas → asset **sin asignar**.
- **RF-INT-06**: el profesor puede **cancelar** una subasta en curso; nadie recibe el asset y todas las pujas se liberan íntegramente.

### 4.2 Recompensas (Sección 9 del PRD — define qué vende Mercado)

- **RF-REC-01**: las recompensas de un curso solo se usan en ese mismo curso, sin excepción.
- **RF-REC-02**: las insignias son cosméticas/prestigio, no se usan ni consumen.
- **RF-REC-03**: insignias = cosmético sin efecto mecánico. Equipamiento = único tipo de recompensa con efecto mecánico.
- **RF-REC-04**: "Desafío de recuperación de vida" — solo para alumnos con 0 vidas, no consume vidas, reintentable indefinidamente, obligatorio para seguir tomando desafíos.
- **RF-REC-05**: el equipamiento con efecto mecánico se consume al usarse (uso único). **Desde la decisión #13, modelar esto (estado por instancia) es responsabilidad de Grupo 12, no de Mercado.** Mercado solo define el efecto/contrato de nombre al armar el catálogo (decisión #11).
- **RF-REC-06**: el profesor carga un pool de desafíos de recuperación de vida; el sistema elige uno al azar priorizando los no resueltos.

### 4.3 Configuración — catálogo de parámetros económicos (Sección 4.1 del PRD)

Todo el catálogo de economía es configuración global **exclusiva de ADMIN** (RF-CFG-04/05).

| ID | Parámetro | Default |
|---|---|---|
| **PAR-06** | Precio en monedas de 1 vida | **300** |
| **PAR-07** | Precio de equipamiento con efecto mecánico | **500** |
| **PAR-12** | Vidas iniciales / máximo vigentes por curso | **3 / 3** |

- **RF-CFG-06**: un cambio de parámetro rige solo hacia adelante — toda orden de compra guarda el **precio con el que se ejecutó** (snapshot).

### 4.4 Requerimientos no funcionales relevantes

- **RF-NFR-01**: borrado lógico en todas las entidades de Mercado (catálogo, órdenes) — sin excepción.
- **RF-NFR-03**: la plataforma soporta 120 usuarios/sesiones concurrentes. Para Mercado, el pico real es el cierre simultáneo de una subasta.
- **RF-NFR-06 / Tabla 9**: catálogo de intercambio (compra directa) = solo Escritorio; subastas (seguimiento y puja) = Escritorio y Móvil.
- **RF-CUR-08/09**: curso archivado = solo lectura.
- **RF-NOT-02**: "nuevos intercambios disponibles" es evento notificable.
- **RF-TUR-04**: el Guided Tour incluye un tour de canje obligatorio.

### 4.5 Gap detectado en el PRD

El registro de riesgos oficial (Tabla 12, RSK-01 a RSK-14) **no incluye ningún riesgo específico de la economía de intercambio** (fraude en subastas, doble gasto, condiciones de carrera). El equipo lo compensa exigiendo pruebas de concurrencia en su propia Definition of Done, pero vale la pena registrarlo aparte.

---

## 5. Modelo de dominio (Mercado, sin inventario)

Entidades candidatas, todas con `cursoCohorteId` obligatorio y baja lógica (RF-NFR-01). Dueño exclusivo: Tema 09.

| Entidad | Atributos clave |
|---|---|
| **ItemTemplate** | `id`, `tipo` (SHIELD \| BOOST_XP \| BOOST_COINS \| LIFE), `parametrosConfigurables` (rango de precio, magnitud del efecto, cargas, etc.), `efecto` (contrato de nombre con Tema 10/Grupo 12). Set cerrado de tipos, **no de ítems concretos** (decisión #14). |
| **OfertaCatalogo** | `id`, `cursoCohorteId`, `itemTemplateId`, `precioMonedas` (elegido por el profesor dentro del rango del template), `configuracion` (magnitud, cargas, desafíos aplicables), `estado` — **sin campo de stock** (decisión #6: disponibilidad siempre ilimitada mientras esté activa). Ya no existen tiers fijos: el tier es el resultado de la configuración que elige el profesor. |
| **Orden** | `id`, `cursoCohorteId`, `alumnoId`, `ofertaId`, `precioAplicado` (snapshot, RF-CFG-06), `holdId` (nombre de campo unificado — ver Sección 12-D, contrato #3), `idempotencyKey`, `estado`, `creadaEn` |
| **Subasta** | `id`, `cursoCohorteId`, `itemTemplateId`, `profesorId`, `inicio`, `fin`, `pujaMinima`, `estado`, `version` (bloqueo optimista) |
| **Puja** | `id`, `subastaId`, `alumnoId`, `monto`, `holdId`, `estado`, `creadaEn` |

**Lo que NO es de Mercado:** saldo de monedas (Grupo 12/Banco), **inventario/mochila del alumno** (Grupo 12/Banco, decisión #13), vidas/XP (Tema 10), matrícula (Tema 02), parámetros (Tema 12), identidad (Tema 01).

**Relaciones:** ItemTemplate 1→0..\* OfertaCatalogo; OfertaCatalogo 1→0..\* Orden; ItemTemplate 1→0..\* Subasta; Subasta 1→0..\* Puja. Ya no hay relación con `ItemInventario` — la acreditación final del ítem es un contrato externo con Grupo 12 (ver Sección 9.1).

### Preguntas de modelado que ya no aplican a Mercado

- ¿La vida entra al inventario como instancia, o se acredita directo en Tema 10? — **Deja de ser pregunta de Mercado**: con inventario en Grupo 12, esto se resuelve entre Grupo 12 y Tema 10.
- ¿`OfertaCatalogo` persiste su propio precio o siempre lee el vigente de Backoffice? — Resuelto indirectamente por RF-CFG-06 + decisión #9: la `Orden` guarda el precio como snapshot (obligatorio); `OfertaCatalogo` puede refrescar contra el endpoint sincrónico de Backoffice al momento de mostrarse, sin necesidad de persistir un precio propio de forma duradera.

---

## 6. Máquinas de estado

### Orden (compra directa)
```
CREADA → HOLD_SOLICITADO → HOLD_CONFIRMADO_BANCO → CONFIRMADA
                ├─→ RECHAZADA_SALDO       (saldo insuficiente en Banco)
HOLD_CONFIRMADO_BANCO ─┼─→ CANCELADA      (falla técnica en la acreditación del ítem → libera el hold)
                └─→ EXPIRADA              (TTL del hold vencido)
```
Nota: ya no existe un paso "stock" ni "crear ItemInventario" local — el hold se pide directo a Banco, y la acreditación del ítem se pide y corrobora contra Grupo 12 antes de confirmar el débito (decisión #8 + #13).

### Subasta

**Enum unificado** (la auditoría encontró 3 enumeraciones distintas entre `diagramas-mercado.md`, `02-matriz-fallos...md` y `03-contratos...md` — esta es la canónica a partir de ahora):

```
DRAFT → SCHEDULED → OPEN (NO_BIDS ↔ ACTIVE_BIDS)
                       ├─→ CANCELLED
                       └─→ CLOSING_IN_PROGRESS
                             ├─ EVALUATING_WINNER
                             ├─ CREDITING_ITEM        [antes: CREDITING_INVENTORY — ahora es una llamada a Grupo 12, no una inserción local]
                             ├─ CONFIRMING_LEDGER     [orden invertido respecto a una fuente vieja — ver nota]
                             ├─ RELEASING_LOSERS
                             ├─ MARKED_DESERTED (sin pujas)
                             └─→ FAILED_SETTLEMENT (AWAITING_MANUAL_OR_CRON_RETRY)
                                     └─→ vuelve a CLOSING_IN_PROGRESS si el reintento tiene éxito
                       → CLOSED
```

**Nota de orden (corrige una contradicción real encontrada en la auditoría):** `CREDITING_ITEM` va **antes** que `CONFIRMING_LEDGER`, no al revés. Una fuente vieja (`Mercado/Subastas/02-matriz-fallos-resiliencia-y-soluciones.md`) tenía el orden invertido (confirmaba el débito antes de acreditar el ítem), lo cual iba contra la decisión #8 y contra lo que el propio `diagramas-mercado.md` mostraba en su diagrama de secuencia. El orden correcto, consistente con la compra directa: corroborar que el ítem se puede entregar/acreditar en Grupo 12 → recién ahí confirmar el débito. Así el caso de "pagó pero no recibió el ítem" deja de ser un escenario normal a compensar — solo ocurre por falla técnica genuina después de corroborar.

### Puja
```
ACTIVA → SUPERADA (liberada, otro alumno pujó más alto)
ACTIVA → GANADORA (al cierre, era la mayor)
ACTIVA → LIBERADA (cierre sin ganar, o cancelación de la subasta)
```

### ItemInventario — YA NO ES DE MERCADO

Esta máquina de estados (y toda la entidad) pasó a ser responsabilidad de Grupo 12 (Banco) por la decisión #13. Se documenta en su lugar el contrato de acreditación en la Sección 9.1.

---

## 7. Compra directa — flujo end-to-end (redefinido tras la decisión #13)

### Regla general (decisión #8)

El orden de operaciones es **reservar → acreditar/corroborar el ítem contra Grupo 12 → confirmar el débito recién si la acreditación ya se corroboró**. Los rechazos por **regla de negocio** no necesitan compensación; una **falla técnica** durante la confirmación sí necesita liberar la reserva.

### Secuencia (redefinida — ya no crea `ItemInventario` localmente)

```
Alumno → Gateway → Mercado: POST /mercado/ordenes {ofertaId, idempotencyKey}
Mercado valida: cohorte activa, oferta activa, precio vigente
  (precio: evento PARAMETRO_ACTUALIZADO + caché local, o GET sincrónico a Backoffice si hace falta el dato al instante — decisión #9)
Mercado → Gateway → Banco: HOLD_CREATE_REQUESTED {holdId, orderId, studentId, courseId, amount, currency, orderType:"DIRECT_PURCHASE"}
Banco → Mercado: HOLD_CREATED {holdId, orderId, studentId, courseId, amount, currency, status:"PENDING", expiresAt}
Mercado → Gateway → Grupo 12 (Banco/Inventario): ITEM_PROVISION_REQUESTED {orderId, holdId, studentId, courseId, itemPayload}
Grupo 12 acredita el ítem en su propio inventario y responde:
  ITEM_PROVISIONED {orderId, holdId, studentId, courseId, inventoryItemId, itemType, state}
  o ITEM_PROVISION_FAILED {orderId, holdId, reason} → Mercado pide HOLD_RELEASE_REQUESTED {holdId, reason} → Orden CANCELADA
Mercado → Gateway → Banco: HOLD_CONFIRM_REQUESTED {holdId, orderId, studentId, courseId, amount}
Banco → Mercado: HOLD_CONFIRMED {holdId, orderId, studentId, amountDebited, ledgerEntryId, status:"COMMITTED"}
Orden = CONFIRMADA (+ outbox) → 201 al alumno
Mercado publica PURCHASE_CONFIRMED al bus
```

> **Oportunidad de simplificación a coordinar con Grupo 12 (no unilateral):** ahora que Grupo 12 es dueño tanto del ledger como del inventario, el paso de "hold → acreditar ítem → confirmar débito" podría colapsar en una sola saga interna de Grupo 12, con Mercado solo esperando un evento final (`ORDER_SETTLED` o similar) en vez de orquestar 3 idas y vueltas. Esto **requiere acuerdo bilateral con Grupo 12** — no se asume acá, se deja anotado como propuesta a plantear formalmente (mismo tratamiento que la decisión #7, que también quedó pendiente de comunicación formal con otro equipo).

### Variantes de falla

| Caso | Tratamiento |
|---|---|
| Saldo insuficiente | Banco rechaza en el paso de `HOLD_CREATE_REQUESTED` → orden `RECHAZADA_SALDO`. Nada que compensar. |
| Rechazo por regla de negocio | Se corrobora antes de confirmar el débito (decisión #8) → no se confirma, no hace falta compensación. |
| Falla técnica en la acreditación del ítem (Grupo 12) | `HOLD_RELEASE_REQUESTED(holdId)` → orden `CANCELADA`. |
| Timeout al confirmar | Reintento con la misma `idempotencyKey`; si persiste, el hold expira por TTL y un job de reconciliación cierra la orden. |
| Monedas de otro curso | Rechazo inmediato, sin llegar a pedir el hold (RF-INT-04). |

### Contrato técnico de referencia

`Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md` sigue siendo la referencia técnica más madura para el mecanismo de reserva/confirmación (SSE, envoltura de eventos, idempotencia) — **salvo** en los puntos que este documento redefine: el modelo de stock (ya no aplica, decisión #6) y quién inserta el ítem (ahora Grupo 12, no Mercado, decisión #13). Los nombres de evento de Banco (`HOLD_*`) siguen siendo el contrato bilateral vigente y no se renombran.

---

## 8. Subastas — diseño detallado

Épica E-07. La compra directa es una transacción de 3-5 segundos; una subasta es un proceso asíncrono de larga duración (horas/días), con pico de concurrencia estimado en ~120 sesiones simultáneas en el minuto final.

### 8.1 Modelo de holds — Opción 1, definitiva (no transicional)

**Confirmado (decisión #3): la Opción 1 (Hold Escrow Total) es la arquitectura final, no un paso intermedio hacia la Opción 2.** Banco retiene el 100% de cada puja de cada participante hasta el cierre; al cierre se confirma la ganadora y se liberan todas las demás en batch.

> **Corrección de auditoría:** `Mercado/Subastas/01-analisis-opciones-arquitectura.md` (Sección 6, "Dictamen del Analista Senior") y `02-matriz-fallos-resiliencia-y-soluciones.md` (solución al Error 4) todavía describen la Opción 2 (Leader-Only Floating Hold) como la "arquitectura objetivo" hacia la que Mercado debería evolucionar. **Eso queda descartado por la decisión #3.** La Opción 2 se mantiene documentada únicamente como análisis histórico de por qué se evaluó y no se eligió — no como roadmap. Los payloads SSE específicos de Opción 2 (`{status:"LEADER"}`/`{status:"OUTBID"}`) quedan huérfanos y no deben implementarse.

### 8.2 Los 5 errores críticos y sus soluciones (vigentes, sin cambios)

1. **Desincronización de expiración (TTL Drift):** Grace Period de +30 min; el cierre lo decide Mercado explícitamente.
2. **Doble cierre:** bloqueo optimista (`@Version`) + coordinación distribuida (ShedLock o partición por `auctionId`).
3. **Vidas subastadas rompen el tope de Tema 10:** las vidas no son subastables por definición de producto.
4. **Tormenta de liberaciones al cierre:** Batch Release, evento consolidado `AUCTION_CLOSED` con lista de `holdIds`.
5. **Desconexión móvil tras ofertar:** `X-Idempotency-Key` + reconexión SSE con `Last-Event-ID`.

### 8.3 Matriz de fallos por microservicio (actualizada — inventario ahora es Grupo 12)

| Microservicio | Momento | Impacto | Solución |
|---|---|---|---|
| Banco | Puja inicial (`HOLD_CREATE`) | Reserva no se asienta | Rechazo limpio, sin impacto contable |
| Banco | Cierre (`HOLD_CONFIRM`) | Débito definitivo no registrado | `CLOSING_PENDING_SETTLEMENT` + Outbox con backoff exponencial |
| Banco | Liberación (`HOLD_RELEASE`) | Perdedores no desbloqueados | `market_pending_refunds` + cronjob de re-emisión |
| Mercado | Crash del pod durante cierre | Ganador elegido pero comandos no enviados | `AuctionRecoveryService` al arrancar, idempotente (`orderId = auctionId`) |
| Kafka | Caída de clúster/partición | Comandos no entregados | Transactional Outbox Pattern (tabla `market_outbox`) |
| **Grupo 12 (antes "Inventario")** | Fallo al acreditar el ítem ganador | El hold del ganador todavía no se confirmó — **no hay nada que compensar** si se respeta el orden correcto (Sección 6): se reintenta la acreditación antes de pedir `HOLD_CONFIRM_REQUESTED`. |

### 8.4 Doctrina de idempotencia en 3 capas (vigente, sin cambios)

1. `commandId = SHA-256(auctionId + studentId + bidSequenceNumber)`.
2. Tabla de deduplicación en Banco (`processed_commands`).
3. Bloqueo optimista en Mercado (`@Version` sobre `MarketAuction`).

Envoltura estándar de todo evento: `{eventId, eventType, timestamp, producer, payload}`.

### 8.5 Contratos de eventos de la saga de subasta

Bilaterales con Banco (`HOLD_*`, no se renombran):

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
| `HOLD_REJECTED` | Banco → Mercado | `bank.holds.events` | **Nuevo — antes sin especificar.** `commandId, auctionId, studentId, reason` |

Propios de Mercado — **renombrados a inglés** (decisión #4, corrige la auditoría):

| Nombre viejo (en español, en las fuentes) | Nombre confirmado | Tópico | Consumidores |
|---|---|---|---|
| `SUBASTA_ADJUDICADA` | **`AUCTION_AWARDED`** | `market.auctions.events` | Notificaciones (11), Backoffice (12), Cursos (02) |
| `OFERTA_SUPERADA` | **`BID_OUTBID`** | canal SSE / `notifications.alerts` | Notificaciones (11), Web & Móvil |
| `SUBASTA_CERRADA` | **`AUCTION_CLOSED`** | `market.auctions.events` | Notificaciones (11), Grupo 12 |

**Nuevo contrato — acreditación del ítem al ganador** (antes sin especificar formalmente en ningún doc de subastas):

| Evento | Dirección | Payload |
|---|---|---|
| `ITEM_PROVISION_REQUESTED` | Mercado → Grupo 12 | `auctionId, winnerStudentId, courseId, itemPayload` |
| `ITEM_PROVISIONED` | Grupo 12 → Mercado | `auctionId, winnerStudentId, courseId, inventoryItemId, itemType, state` |
| `ITEM_PROVISION_FAILED` | Grupo 12 → Mercado | `auctionId, winnerStudentId, reason` |

Particionamiento Kafka: `partitionKey = auctionId` en todos los tópicos de subasta.

**Convención de nombres de productor/consumidor (corrige la auditoría):** usar siempre `team-09-market` / `team-08-bank` y grupos de consumidor `market-*-group` / `bank-*-group`, en inglés — nunca `tema-09-mercado`/`mercado-*-group` (esa variante en español apareció solo en `03-contratos-eventos-e-idempotencia.md` y queda descartada).

### 8.6 Directivas de implementación

1. Nunca emitir a Kafka fuera de una transacción de BD local — tabla `outbox_events`.
2. TTL con gracia de +30 min; el cierre es exclusivo de Mercado.
3. No permitir subastar consumibles con tope — solo cosméticos/equipables sin límite rígido.
4. Alerta temprana: métrica `auctions_closing_pending_seconds`, crítica si > 120s en `CLOSING_IN_PROGRESS`.

---

## 9. Contratos de integración por dependencia

### 9.1 Banco / Grupo 12 — dependencia crítica bloqueante (ahora también dueño de inventario)

Protocolo (saga de pasos, no es "descuento y después entrego"):
```
1. HOLD_CREATE_REQUESTED(alumno, curso, monto, idempotencyKey) → HOLD_CREATED | HOLD_REJECTED
2. ITEM_PROVISION_REQUESTED → ITEM_PROVISIONED | ITEM_PROVISION_FAILED   (Grupo 12 acredita en su propio inventario, ya no Mercado)
3. HOLD_CONFIRM_REQUESTED → HOLD_CONFIRMED (débito efectivo en el ledger)
   o HOLD_RELEASE_REQUESTED → HOLD_RELEASED (devolución sin costo)
```
Sincrónico vía Gateway para comandos con respuesta inmediata, asincrónico vía Kafka para hechos consumados. Envoltura estándar de 5 campos. Broker: Kafka, semántica *at-least-once*, `partitionKey = studentId` para compra directa.

> **Partición inconsistente detectada en la auditoría:** el contrato de subastas usa `partitionKey = auctionId` para el mismo tópico `bank.holds.commands` donde compra directa usa `studentId`. Esto es intencional (distinto caso de uso), pero **hay que documentarlo explícitamente como partición dual por `orderType`**, no dejarlo como una inconsistencia sin explicar — Banco necesita saber que las garantías de orden FIFO son por auction, no por alumno, en el caso de subastas.

Puntos que **siguen sin acordar formalmente con Grupo 12** (no son decisiones de Mercado en solitario): la oportunidad de simplificación de la Sección 7 (colapsar hold+acreditación+confirmación en una sola saga interna de Grupo 12); códigos de error diferenciados completos; parámetros exactos de reintento/timeout (ver Sección 12-C).

### 9.2 Cursos y Matrícula (Tema 02)

- **Consulta síncrona**: `GET /api/v1/courses/{courseId}/students/{studentId}/enrollment-status` → `{courseId, studentId, enrolled, status, enrolledAt}`. Si `enrolled:false` → `403 {NOT_ENROLLED_IN_COURSE}`.
- **Tópico `courses.lifecycle`**:
  - `COURSE_ARCHIVED` (partitionKey `courseId`): Mercado bloquea nuevas órdenes/compras/subastas de ese curso.
  - `STUDENT_UNENROLLED` (partitionKey `studentId`): **decisión #10 — mismo tratamiento que `COURSE_ARCHIVED`**, solo lectura para ese alumno en esa cohorte, sin acción adicional de Mercado (el estado de sus ítems ya acreditados es responsabilidad de Grupo 12).
- **Decisión #7:** Cursos debe bloquear el archivado de un curso mientras tenga subastas activas — pendiente de comunicación formal con ese equipo.
- La tabla `student_inventory` **ya no es de Mercado** (era del modelo viejo) — si existe, es responsabilidad de Grupo 12 desde la decisión #13.

### 9.3 Identidad y Gateway (Tema 01)

- Cabeceras inyectadas: `X-User-Id`, `X-Roles`, `X-User-Email`, `X-Forwarded-For`. Mercado nunca valida JWT.
- Roles (RBAC local): `ROLE_STUDENT` (consulta catálogo, compra, puja), `ROLE_PROFESSOR` (curaduría de ofertas por cohorte, lanza/cancela subastas), `ROLE_ADMIN` (auditoría global).
- Errores: `401 {INVALID_OR_EXPIRED_TOKEN}` (Gateway), `403 {ROLE_NOT_PERMITTED}` (Mercado).

### 9.4 Motor de Desafíos (Tema 03) — YA NO involucra a Mercado

**Decisión #11 (resuelta el 18/09/2026): el consumo de equipamiento sale completo del alcance de Mercado.** El contrato de consulta/consumo de ítems equipados pasa a ser exclusivamente entre Motor de Desafíos (Tema 03) y Grupo 12 (Banco). Mercado no expone `GET /api/v1/market/inventory/...`, no procesa `DESAFIO_RESUELTO`, y no publica `ITEM_CONSUMED`.

Lo único que Mercado sigue aportando: el **contrato de nombre del efecto** (`verb`/`actionParams`, ej. `ABSORB_FAILURE`, `XP_MULTIPLIER`, `COIN_MULTIPLIER`) al definir un `ItemTemplate` en el catálogo (Sección 5) — ese nombre de efecto es lo que Grupo 12 y Tema 10 usan para aplicar la mecánica, pero la consulta y el descuento de cargas ya no pasan por Mercado.

### 9.5 Backoffice (Tema 12)

- **Asíncrono**: tópico `backoffice.parametros`, evento `PARAMETRO_ACTUALIZADO` → Mercado invalida caché local, hot-reload sin downtime.
- **Sincrónico (nuevo — decisión #9):** `GET /api/v1/backoffice/parametros/{id}` (rol `PROFESSOR`/`ADMIN`), para pedir PAR-06/PAR-07 al instante en vez de depender solo del evento + caché. Se usa cuando Mercado necesita el precio vigente con la mayor frescura posible (ej. al mostrar el catálogo o al validar una orden).
- `GET /api/v1/market/admin/metrics?courseId=` (rol `PROFESSOR`), snapshot agregado, frescura máxima 15 minutos.

### 9.6 Notificaciones (Tema 11)

Dueño del contrato de eventos de toda la plataforma. Consume los eventos de Mercado (Sección 10) para notificar: nuevo ítem en catálogo, subasta que arranca, puja superada, subasta ganada/perdida/cancelada.

### 9.7 Ranking / Roadmap y Progreso (Tema 10)

- Acreditación de vidas y aplicación del efecto de equipamiento: ahora es un flujo entre Grupo 12 y Tema 10, sin mediación de Mercado (decisión #13).
- La validación de tope de vidas (PAR-12) al comprar sigue la decisión #8: reservar → corroborar la acreditación con Tema 10/Grupo 12 → confirmar el débito solo si se aceptó.

---

## 10. Catálogo de eventos que Mercado publica/consume (actualizado)

Convención (decisión #4): **MAYÚSCULAS_SNAKE_CASE, en inglés**, para todo evento que Mercado diseñe y controle.

| Evento | Tópico | Notas |
|---|---|---|
| `CATALOG_OFFER_PUBLISHED` | `market.catalog.events` | Nueva oferta de catálogo publicada por el profesor |
| `PURCHASE_CONFIRMED` | `market.orders.events` | Compra directa confirmada |
| `AUCTION_OPENED` | `market.auctions.events` | Subasta abierta |
| `AUCTION_CLOSED` | `market.auctions.events` | Renombrado desde `SUBASTA_CERRADA` (decisión #4) |
| `AUCTION_CANCELLED` | `market.auctions.events` | Subasta cancelada por el profesor |
| `AUCTION_AWARDED` | `market.auctions.events` | Renombrado desde `SUBASTA_ADJUDICADA` |
| `BID_OUTBID` | SSE / `notifications.alerts` | Renombrado desde `OFERTA_SUPERADA` |

**Eliminado (decisión #13): `ITEM_CONSUMED` — Mercado ya no lo publica.** Si Grupo 12 necesita un evento equivalente para su propio dominio, es una decisión de ellos, no de Mercado.

Eventos que Mercado **consume** (no se renombran): `COURSE_ARCHIVED`, `STUDENT_UNENROLLED` (Tema 02), `PARAMETRO_ACTUALIZADO` (Tema 12), `HOLD_CREATED`/`HOLD_INCREASED`/`HOLD_CONFIRMED`/`HOLD_RELEASED`/`HOLD_REJECTED` (Banco), `ITEM_PROVISIONED`/`ITEM_PROVISION_FAILED` (Grupo 12).

Regla de diseño: un evento es un hecho consumado, nombre en pasado, sin pedir respuesta. Si hace falta respuesta para continuar, es sincrónico por el Gateway.

---

## 11. Backlog para Sprint 1 (ratificado — decisión #12)

**Confirmado el 18/09/2026: Sprint 1 sigue siendo M-00 + M-01 + M-02**, ajustadas al recorte de inventario (decisión #13) y al catálogo por plantillas (decisión #14).

### Definition of Done (11 puntos, sin cambios)

Criterios de aceptación con escenarios de falla (formato Gherkin) · code review aprobado · toda comunicación entre microservicios pasa por el Gateway · borrado lógico respetado · al menos un escenario de concurrencia probado si la historia toca saldo · formato de eventos conforme al contrato acordado · probado manualmente por alguien distinto de quien codeó · sin errores/warnings en consola durante demo · endpoints nuevos documentados antes de cerrar · tarjeta de Taiga reflejando estado real · aprobación del PO antes de "Done".

### Capacidad efectiva del equipo

| Indicador | Valor |
|---|---|
| Duración del Sprint | 10 días hábiles |
| Integrantes | 11 |
| Capacidad teórica | 880 h |
| Ceremonias Scrum | 71,5 h |
| **Capacidad efectiva** | **323,4 h (~37% de un sprint normal)** |

### Épica M-00 — Spike técnico: contrato de reserva/confirmación con el Banco

Estado: **resuelto**. Reservar→confirmar/liberar (Sección 9.1), idempotencia por `idempotencyKey`, subastas usan un contrato de holds más elaborado (Sección 8), sin stock/límite de compras (decisión #6). Sigue abierto solo el canal de notificación del resultado y los parámetros exactos de reintentos/timeout con Banco (Sección 12-C).

### Épica M-01 — Catálogo de ítems (reescrita — decisión #14)

- **HU-01.1**: Como profesor, armo el catálogo de mi curso-cohorte eligiendo un `ItemTemplate` (tipo) y configurando sus parámetros permitidos (precio dentro de rango, magnitud del efecto, cargas) — ya no cargo un ítem "desde cero" ni elijo un tier fijo.
- **HU-01.2**: Como alumno, veo la lista de ofertas de catálogo activas de mi curso-cohorte (nombre derivado del template + configuración, precio vigente).
- **HU-01.3**: Como profesor, el catálogo que armo solo es visible/editable para el curso-cohorte correcto.
- Tareas: modelar `ItemTemplate` + `OfertaCatalogo` (sin campo de stock); endpoint de gestión para el profesor (elegir template + configurar); endpoint de listado filtrado por curso-cohorte para el alumno; leer precios por defecto de PAR-06/PAR-07 (evento + endpoint sincrónico, decisión #9); UI de catálogo (alcance escritorio).

### Épica M-02 — Compra directa con reserva contra el Banco (ajustada — decisión #13)

- **HU-02.1** — Confirmar compra y ver resultado:
  - Escenario 1 (éxito): saldo suficiente → hold → Grupo 12 acredita el ítem → confirmación del débito → ítem acreditado en Grupo 12, monedas descontadas.
  - Escenario 2 (falla técnica en la acreditación del ítem): el hold se libera, no se confirma el débito, alumno recupera saldo.
  - Escenario 3 (monedas de otro curso): rechazo, se informa que las monedas no pertenecen a ese curso.
- **HU-02.2**: feedback visual durante la compra (botón deshabilitado + spinner) para evitar doble clic.
- Tareas: flujo hold→acreditación(Grupo 12)→confirmación contra Banco; validar en orden disponibilidad→pertenencia al curso→saldo→moneda del curso correcto; reintentos automáticos con compensación (falla técnica); pruebas de concurrencia sobre compras simultáneas del mismo ítem (ya no hay condición de carrera de stock, decisión #6, pero sí de saldo).

---

## 12. Estado de decisiones

### 12-A. Decisiones confirmadas el 16/09/2026

1. Fase de subastas = Fase 3.
2. Broker de mensajería = Kafka, definitivo para toda la plataforma.
3. Modelo de hold en subastas = Opción 1 (Hold Escrow Total), **definitiva, no transicional**.
4. Nomenclatura de eventos propios de Mercado: MAYÚSCULAS_SNAKE_CASE, en inglés, sin excepción.
5. Origen del equipamiento con efecto mecánico: lo define el profesor, por curso-cohorte.
6. Sin límite de stock: la oferta de catálogo tiene disponibilidad ilimitada mientras esté activa.
7. Archivado de curso con subasta abierta: Cursos debe bloquear el archivado — pendiente de comunicación formal con ese equipo.
8. Validación de reglas de negocio sin compensación: reservar → corroborar → confirmar el débito solo si se corroboró.

### 12-B. Decisiones confirmadas el 18/09/2026 (en esta sesión)

9. **Sincronización de PAR-06/PAR-07:** se agrega endpoint sincrónico además del evento + caché local.
10. **Desmatriculación a mitad de curso:** mismo criterio que curso archivado (solo lectura).
11. **Consumo de equipamiento con efecto mecánico:** sale completo del alcance de Mercado — contrato exclusivo Motor de Desafíos ↔ Grupo 12.
12. **Alcance de Sprint 1:** se ratifica M-00 + M-01 + M-02, ajustadas.
13. **Inventario/mochila del alumno:** sale de Mercado, pasa a Grupo 12 (Banco).
14. **Catálogo por plantillas:** se adopta, reemplaza el catálogo canónico de 16 consumibles con tiers fijos.

**Ya no quedan pendientes genuinos bloqueantes para arrancar Sprint 1.**

### 12-C. Preguntas menores que siguen abiertas (no bloquean Sprint 1, no se trataron en esta sesión)

- Canal de notificación del resultado de una compra y parámetros exactos de reintentos/timeout con Banco (M-00).
- Vencimiento de ítems: ¿por fecha, por cierre de curso, o ambos? Ahora es una pregunta de Grupo 12 más que de Mercado, pero afecta cómo Mercado modela `configuracion` en `OfertaCatalogo` (ej. si hay que guardar una fecha de expiración sugerida).
- Intercambio entre alumnos: ¿entra en algún sprint? Si entra, cuidado con RF-INT-01.
- Gap de riesgos del PRD (Sección 4.5) — vale la pena que el equipo lo agregue a su propio registro.
- La oportunidad de simplificación de la saga de compra directa (Sección 7, colapsar en una sola saga de Grupo 12) — a proponer formalmente, no asumida.

### 12-D. Auditoría de fuentes (18/09/2026) — contradicciones encontradas y ya corregidas en este documento

La relectura completa de `diagramas-mercado.md`, `Mercado/Catalogos/README.md`, `Mercado/Subastas/{README,01,02,03}.md`, `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md` y `README.md` raíz encontró:

1. **Modelo de stock finito** (`StockHold`, `availableStock`, SQL de decremento, `409 OUT_OF_STOCK`) en 4 archivos — contradice la decisión #6. Corregido en Secciones 5 y 7 de este documento; **las fuentes originales todavía no están corregidas**.
2. **Catálogo fijo de ítems concretos** (`ItemBaseTemplate` con 4 tipos cerrados, sin parametrización real) en 3 archivos — contradice la decisión #5 y quedó superado además por la decisión #14. Corregido en Sección 5.
3. **Opción 2 de subastas tratada como arquitectura objetivo** en `01-analisis-opciones-arquitectura.md` y `02-matriz-fallos...md` — contradice la decisión #3. Corregido en Sección 8.1.
4. **3 eventos propios de Mercado en español** (`SUBASTA_ADJUDICADA`, `OFERTA_SUPERADA`, `SUBASTA_CERRADA`) — contradice la decisión #4. Renombrados en Sección 8.5/10.
5. **Orden de liquidación de subastas invertido** (débito antes de acreditar el ítem) en `02-matriz-fallos...md` — contradice la decisión #8. Corregido en Sección 6.
6. **3 enumeraciones distintas de `AuctionStatus`** entre 3 archivos — unificadas en Sección 6.
7. **Convención de nombres `tema-09-mercado`/`mercado-*-group` en español** en `03-contratos-eventos-e-idempotencia.md`, inconsistente con el resto de la plataforma — estandarizado a `team-09-market`/`market-*-group` en Sección 8.5.
8. **Nombres de campo inconsistentes** entre contratos: `holdId` vs `bankHoldId` (se adopta `holdId`), `amountDebited` vs `debitedAmount` (se adopta `amountDebited` para compra directa, `debitedAmount` es el que ya usaba el contrato de subastas — **queda como pendiente menor unificar uno solo, ver 12-C**), `reason` vs `releaseReason` (se adopta `releaseReason`), `COMPENSATED_RELEASED` vs `COMPENSATED_FAILED` (se adopta `CANCELADA` como estado de Orden en este documento, ver Sección 6).

**Pendiente de trabajo aparte (no bloquea Sprint 1):** aplicar estas correcciones físicamente en los 6 archivos fuente listados arriba, o marcarlos explícitamente como superados por este documento. Mientras tanto, este documento es la autoridad para los 22 puntos de las Secciones 12-A, 12-B y 12-D.

---

## 13. Referencias

| Documento | Contenido | Estado tras la auditoría del 18/09 |
|---|---|---|
| `README.md` (raíz) | Índice general del repo y mapa de tópicos Kafka | Desactualizado: modelo de stock (12-D #1), catálogo fijo (12-D #2), falta el mapa de tópicos de subastas |
| `diagramas-mercado.md` | Borradores Mermaid: contexto C4, modelo de dominio, máquinas de estado, secuencias | Desactualizado: `StockHold` (12-D #1), `ItemBaseTemplate` fijo (12-D #2), enum de subasta distinto (12-D #6) |
| `Mercado/Catalogos/README.md` | Spec técnica de catálogo y stock hold | Desactualizado en casi todo su contenido de stock (12-D #1) y plantillas fijas (12-D #2) |
| `Mercado/Subastas/README.md` | Índice del paquete documental de Subastas | Vigente como índice; los 3 docs que enlaza están desactualizados |
| `Mercado/Subastas/01-analisis-opciones-arquitectura.md` | 3 opciones de arquitectura de holds, con dictamen | Desactualizado: recomienda Opción 2 como objetivo (12-D #3) |
| `Mercado/Subastas/02-matriz-fallos-resiliencia-y-soluciones.md` | 5 errores críticos, matriz de fallos, máquina de estados | Desactualizado: propone Opción 2 (12-D #3), orden de liquidación invertido (12-D #5), enum propio (12-D #6) |
| `Mercado/Subastas/03-contratos-eventos-e-idempotencia.md` | Contratos de eventos e idempotencia de subastas | Desactualizado: eventos en español (12-D #4), convención `tema-XX` (12-D #7), enum propio (12-D #6) |
| `PRD-Plataforma-Gamificada-TP.pdf` | Fuente de verdad oficial del producto | Vigente, no tocado por esta auditoría |
| `Sprint0_Propuesta_Mercado.pdf` | DoD del equipo, cálculo de capacidad, épicas M-00/M-01/M-02 con historias Gherkin | Vigente como base; las épicas quedaron reescritas en la Sección 11 de este documento |
| `Comunicacion/Grupo-08-Banco/flujo-comunicacion-banco.md` | Saga completa de compra directa con SSE, contratos de eventos | Desactualizado solo en el modelo de stock (12-D #1) y en quién inserta el ítem (ahora Grupo 12); el resto sigue siendo la referencia técnica principal |
| `Workflow/README.md` | Convenciones de branching y commits del repo completo | Vigente, no relacionado con este contenido |
