# Tema 09 — Mercado · Documento de arranque

**Trabajo Integrador · Programación IV Back End · TUP UTN FRC**
Equipo de 10 personas · Estado: repos vacíos, sprint 0, Scrum con Taiga
Stack declarado: Java (Spring Boot) · Angular · JS

---

## 0. Cómo usar este documento

Está ordenado para leerse en el orden en que conviene decidir las cosas hoy:

1. Qué es realmente su tema y dónde termina (§1, §2)
2. De quién dependen y quién depende de ustedes (§3) — **lo más urgente**
3. Qué preguntas hay que llevarle al PO y a la sesión de integración (§4)
4. Qué modelar y qué diagramas producir hoy (§5)
5. Arquitectura back end: reglas de plataforma y patrones que van a necesitar (§6)
6. Arquitectura front end: las tres propuestas explicadas a fondo (§7)
7. Scrum y Taiga (§8)
8. Cómo repartir 10 personas (§9)
9. Ambiente de trabajo y herramientas (§10)
10. Qué leer, en qué orden (§11)
11. Agenda propuesta para la jornada (§12)

Los IDs entre paréntesis (RF-INT-05, PAR-07, etc.) son del PRD. Úsenlos siempre:
el PRD dice explícitamente que **"los IDs son el contrato"** y que un requerimiento
que no está trazado a algo, no está hecho. Cada historia en Taiga debería citar los
RF que cubre.

---

## 1. El sistema en una página (para ubicarse)

Plataforma de e-learning gamificada. Cada curso es un roadmap. El alumno resuelve
desafíos teóricos y prácticos, gana **XP, monedas, vidas, insignias y equipamiento**,
compite en un ranking por curso, y es asistido por IA con reglas pedagógicas estrictas.

Cuatro ideas que definen todo el sistema y que conviene tener grabadas:

- **El curso-cohorte es el contexto de todo.** Casi ninguna entidad existe fuera de un
  curso. Las recompensas se usan **solo** en el curso donde se obtuvieron (RF-REC-01,
  RF-INT-04). No existe saldo global ni inventario global. Si modelan una entidad sin
  la clave de cohorte, después no hay forma de acotarla sin migrar datos.
- **Dos monedas conceptuales, no intercambiables.** El XP mide progreso académico y
  **no se gasta**; las monedas son poder de compra. Nunca se convierte una en otra
  (RF-INT-01/02).
- **No hay borrado físico** de nada, salvo el chat social (RF-NFR-01). Todo es baja
  lógica.
- **Nada se resuelve por llamada directa entre microservicios.** Todo sincrónico pasa
  por el API Gateway; todo lo asincrónico va por el bus de eventos.

El reparto son 12 temas. Ustedes son el **Tema 09 — Mercado**.

---

## 2. Alcance del Tema 09 — Mercado

### 2.1 Lo que la propuesta de arquitectura les pide

| Prioridad | Ítems |
|---|---|
| **Pedido para empezar** | Catálogo · Compra contra reserva del banco · Inventario del alumno con alcance por curso · Consumo de ítems |
| **Para más adelante** | Subastas con ventana temporal · Acceso móvil a subastas · Vencimiento de ítems |
| **Podría ser** | Intercambio entre alumnos · Ítems por temporada · Catálogo configurable por curso |

Nota textual del documento: *"Es el tema más liviano del reparto: los extras son el
mecanismo previsto para equilibrarlo."* **Esto es una señal directa para ustedes:**
con 10 personas, el núcleo (catálogo + compra + inventario + consumo) no alcanza para
llenar el cuatrimestre. Ver §9 para qué hacer con esa capacidad sobrante.

### 2.2 Los requerimientos del PRD que les tocan

**Sección 10 — Sistema de intercambio (su núcleo):**

- **RF-INT-01**: las monedas solo se canjean por **vidas o equipamiento**; nunca al
  revés; nunca se obtienen monedas por intercambio. → El catálogo tiene exactamente
  dos familias de ítems en el MVP.
- **RF-INT-02**: solo las monedas son intercambiables (insignias, XP y vidas no se
  canjean directamente).
- **RF-INT-03**: dos modalidades — **compra directa** (precio fijo) y **subasta**.
- **RF-INT-04**: las monedas usadas deben pertenecer al mismo curso del intercambio.
- **RF-INT-05**: reglas de subasta:
  - el PROFESOR lanza el asset y define duración y (opcional) puja mínima;
  - al pujar, las monedas quedan **bloqueadas/reservadas** (no disponibles para compra
    directa ni otras subastas) hasta el cierre;
  - el pujador puede **aumentar** su oferta, nunca retirarla ni bajarla;
  - al cierre: al ganador se le descuentan las monedas y recibe el asset; a los demás
    se les liberan las reservas sin costo;
  - subasta sin pujas → asset sin asignar.
- **RF-INT-06**: el profesor puede **cancelar** una subasta en curso; nadie recibe el
  asset y todas las pujas se liberan íntegramente.

**Sección 9 — Recompensas (define qué venden):**

- **RF-REC-01**: recompensas de un curso solo se usan en ese curso, sin excepción.
- **RF-REC-03**: insignias = cosmético/prestigio **sin efecto mecánico**;
  **equipamiento = único tipo de recompensa con efecto mecánico** (ej. escudo que
  evita perder una vida).
- **RF-REC-05**: el equipamiento con efecto mecánico **se consume al usarse** (uso
  único). → Su inventario necesita estado por instancia, no solo un contador.
- **Vidas**: cantidad inicial y máximo vigente según **PAR-12** (default 3/3). Si el
  máximo de vidas vigentes es 3, **comprar una vida cuando ya tenés 3 tiene que
  fallar o estar deshabilitado**. Esa validación es del dueño de vidas (Tema 10), no
  de ustedes — pero el mercado tiene que consultarla o manejar el rechazo.

**Parámetros globales que consumen pero no administran (Tema 12 los administra):**

- **PAR-06**: precio en monedas de 1 vida = 300 (default de referencia).
- **PAR-07**: precio de equipamiento con efecto mecánico = 500.
- **RF-CFG-04/05**: son configuración global de ADMIN. **El profesor no puede
  sobreescribirlos.** No los hardcodeen: se leen del Tema 12.
- **RF-CFG-06**: un cambio de parámetro **rige solo hacia adelante**. → Toda orden de
  compra debe guardar el **precio con el que se ejecutó** (snapshot), no una
  referencia al parámetro vigente.

**Requerimientos no funcionales que los tocan:**

- **RF-NFR-06** (tabla 9): en móvil, **"Subastas: seguimiento y puja" está habilitado**;
  **"Catálogo de intercambio (compra directa)" NO lo está**. El motivo está escrito:
  la subasta tiene ventana temporal y un alumno no puede quedar fuera por no estar
  frente a la compu. Cuando alguien entra desde móvil a algo no habilitado, la
  plataforma debe **informarlo explícitamente** ("esta sección requiere una
  computadora"), no degradarse ni fallar.
- **RF-NFR-01**: baja lógica en todas sus entidades.
- **RF-NFR-03**: 120 usuarios / 120 sesiones concurrentes. Para ustedes el pico no es
  el catálogo: es el **cierre simultáneo de una subasta**.
- **RF-CUR-08/09**: curso archivado = solo lectura, sin nuevas acciones. → ¿Qué pasa
  con una subasta abierta al archivar? (ver §4).
- **RF-NOT-02**: "nuevos intercambios disponibles" es un evento notificable → tienen
  que publicar eventos para el Tema 11.
- **RF-TUR-04**: el Guided Tour incluye un **tour especial de canje** que obliga al
  alumno a canjear monedas por un beneficio dentro del curso. → **El mercado es parte
  del onboarding**: sin catálogo funcionando, el tour no cierra. Es una dependencia
  que probablemente el Tema 01 todavía no vio.

### 2.3 Tensión de alcance que hay que resolver con el PO

El PRD (Sección 2 y Sección 18) pone **insignias, equipamiento y sistema de
intercambio por compra directa en Fase 2**, y **subastas en Fase 3** — es decir,
**fuera del MVP**. La propuesta de arquitectura, en cambio, les pide catálogo, compra,
inventario y consumo como "pedido para empezar".

No es una contradicción fatal (el TPI no es el MVP del producto), pero **define si en
el sprint 1 son un equipo bloqueante o un equipo que puede trabajar con mocks**.
Pregunta explícita para el PO en §4.

---

## 3. Mapa de dependencias — su trabajo real

El Mercado tiene poco dominio propio y mucha conversación. Ordenado por criticidad:

### 3.1 Tema 08 — Banco (**dependencia crítica, bloqueante**)

El Banco es **dueño exclusivo del ledger y de los saldos**. Ustedes **nunca** guardan
saldo de monedas. Su "pedido para empezar" incluye **Reservas**, justamente para
ustedes.

El protocolo obligatorio está escrito en la lámina 5: la compra **no puede ser
"descuento y después entrego"**. Es:

```
1. reservar(alumno, curso, monto, idempotencyKey) -> reservaId | RECHAZADA(saldo insuficiente)
2. (mercado entrega el ítem / cierra la orden)
3. confirmar(reservaId)  -> descuento efectivo en el ledger
   o liberar(reservaId)  -> devolución sin costo
```

Lo que tienen que acordar **hoy o mañana** con el Tema 08:

- Nombre, forma y semántica de las tres operaciones (¿REST por el gateway? ¿cuál es el
  contrato exacto?).
- **Idempotencia**: si el mercado reintenta `reservar` por un timeout, no puede
  reservar dos veces. Clave de idempotencia provista por el mercado.
- **Expiración de reserva**: ¿la reserva caduca sola? ¿Quién la libera si el mercado se
  cae entre el paso 1 y el 3? (Respuesta razonable: TTL del lado del Banco + job de
  reconciliación. Hay que acordarlo, no asumirlo.)
- **Alcance por curso**: la reserva viaja siempre con `cursoCohorteId` (RF-INT-04).
- Códigos de error y qué significa cada uno (saldo insuficiente ≠ reserva inexistente
  ≠ reserva ya confirmada).
- **Subastas**: una puja es una reserva más, pero de larga duración (horas o días) y
  **acumulativa cuando el alumno aumenta su oferta**. ¿Se reserva el delta o se libera
  y re-reserva el total? Esto le pega al Banco tanto como a ustedes.

### 3.2 Tema 10 — Roadmap y Progreso (**dependencia crítica**)

El Tema 10 es dueño de **XP, niveles, vidas, logros e insignias**. Ustedes venden
vidas y equipamiento. Choque de fronteras inevitable:

- **Compra de una vida**: ustedes cobran, pero **la vida la acredita el 10**. ¿Es un
  evento (`compra.confirmada`) o una llamada sincrónica? Si es evento y el 10 la
  rechaza porque el alumno ya tiene el máximo (PAR-12), la plata ya se descontó →
  hace falta **validación previa sincrónica** o **compensación**. Decidan y
  documenten.
- **Consumo de equipamiento con efecto mecánico** (RF-REC-05): el escudo evita perder
  una vida. El que "pierde vidas" es el 10. Entonces: ¿el 10 pregunta al 09 "¿este
  alumno tiene un escudo disponible? consumilo"? ¿O el 09 expone `consumir(itemId)` y
  el 10 lo invoca? Nuestra recomendación: **el 09 es dueño del inventario y expone una
  operación de consumo idempotente; el 10 la invoca cuando aplica la regla**. El
  efecto del ítem lo interpreta el 10; la existencia y el estado del ítem los gobierna
  el 09.
- **Quién define el catálogo de efectos**: ustedes venden un `TipoItem` con un
  `efecto` declarado; el 10 sabe qué hacer con cada efecto. El acoplamiento es por
  **contrato de nombres de efecto**, no por código compartido.

### 3.3 Tema 02 — Cursos y Matrícula

- Les da la identidad del **curso-cohorte** y su ciclo de vida (draft → activo →
  archivado).
- Publica el evento **"curso archivado"** → ustedes reaccionan: cerrar/cancelar
  subastas abiertas, liberar reservas, congelar el catálogo, dejar el inventario en
  solo lectura (RF-CUR-09).
- Necesitan resolver contra el 02 la **pertenencia**: ¿este alumno pertenece a esta
  cohorte? (Lámina 7: el token del Tema 01 dice quién sos y qué rol tenés; la
  pertenencia la responde el Tema 02.)

### 3.4 Tema 12 — Backoffice y parámetros

- Administra **PAR-01 a PAR-24** en exclusiva; ustedes son uno de los consumidores
  (PAR-06, PAR-07 y los que definan para su tema).
- Van a necesitar exponerle **contratos de lectura** (el 12 no tiene dominio propio y
  "sin contratos de lectura acordados en el sprint 1 no tiene nada demostrable"):
  por ejemplo, circulante gastado por curso, ítems más comprados, inventario agregado.
- **Frescura máxima de 15 minutos** en los datos del panel → pueden servirlo por
  lectura directa o por proyección; no necesitan tiempo real.

### 3.5 Tema 11 — Social y Notificaciones

- **Define el contrato de eventos de toda la plataforma.** Su decisión los condiciona.
  Vayan a esa conversación temprano con su lista de eventos ya escrita (§5.4).
- Consumen sus eventos para notificar: nuevo ítem en el catálogo, subasta que arranca,
  puja superada, subasta ganada/perdida/cancelada.

### 3.6 Tema 01 — Identidad y Usuarios

- Token: identificador, rol y vigencia. El gateway lo valida; **validar no es
  autorizar**. La regla "solo un alumno matriculado en esta cohorte puede comprar en
  este catálogo" es **regla de negocio de ustedes**, no del gateway.
- Roles que les importan: ALUMNO (compra, puja, consume), PROFESOR (lanza subastas,
  cancela, quizá administra catálogo del curso), ADMIN (todo).

### 3.7 Tema 03 — Motor de Desafíos

- Equipamiento también se obtiene **por desafío**, no solo por compra (tabla 5 del
  PRD). → Su inventario recibe altas desde el 03/10 además de desde el mercado.
  Definan una única puerta de entrada al inventario.

---

## 4. Preguntas abiertas — llevarlas al PO y a la sesión de integración

Priorizadas. Las tres primeras bloquean el diseño.

1. **¿El alcance del TPI es MVP del PRD o incluye Fase 2/3?** Si es MVP estricto, el
   mercado no existe y la asignación de arquitectura manda. Necesitamos que el PO lo
   diga explícitamente. (PRD Sección 2 vs. propuesta Tema 09.)
2. **Compra de vida: ¿validación previa sincrónica con el Tema 10 o compensación?**
   (PAR-12: máximo de vidas vigentes.)
3. **Consumo de equipamiento: ¿quién invoca a quién?** (§3.2)
4. **Desmatriculación a mitad de cuatrimestre** — está listada como *decisión abierta*
   en la propuesta y nos afecta directamente: monedas, inventario y órdenes de un
   alumno quedan acotadas a un curso al que ya no pertenece. ¿Se congela? ¿Se libera?
   ¿Se anula? Afecta al 02, 08, 09 y 10 a la vez.
5. **Archivado de curso con subasta abierta**: ¿el archivado la cancela
   automáticamente (RF-INT-06) o el 02 bloquea el archivado hasta que no haya subastas
   vivas? Precedente en el PRD: RF-IA-34 bloquea el archivado si hay scores diferidos.
6. **¿Quién administra el catálogo?** RF-INT-05 dice que el PROFESOR lanza subastas.
   Para compra directa, RF-CFG-05 dice que los **precios** son de ADMIN. ¿El profesor
   elige qué ítems ofrece en su curso ("catálogo configurable por curso" está en
   *podría ser*), o el catálogo es global y solo el precio es de ADMIN?
7. **Vencimiento de ítems** (está en "para más adelante"): ¿vence por fecha, por cierre
   de curso, o ambos? No está en el PRD; es una definición nueva.
8. **Intercambio entre alumnos** (RF-INT / "podría ser"): ¿entra? Si entra, ¿se
   transfieren ítems, monedas, o ambos? Ojo con RF-INT-01 ("nunca se obtienen monedas
   por intercambio").
9. **¿Nos hacemos cargo de algún transversal sin dueño?** La propuesta lista
   *arquitectura multi-idioma, alcance de la versión móvil y marco de indicadores* como
   **sin asignar**. Siendo el tema más liviano y 10 personas, ofrecerse a tomar uno es
   una jugada fuerte frente al PO. El de **versión móvil** nos toca de cerca (somos el
   único tema con una funcionalidad explícitamente habilitada en móvil: la subasta).

---

## 5. Modelado y diagramas a producir

### 5.1 Entidades candidatas (dueño: Tema 09)

- `ItemDefinicion` (o `ArticuloCatalogo`) — qué se puede obtener: tipo (VIDA,
  EQUIPAMIENTO), efecto declarado, si es consumible, metadata visual.
- `CatalogoCurso` / `OfertaCatalogo` — la definición **publicada en una cohorte** con
  su precio vigente y su disponibilidad (stock limitado o ilimitado).
- `Orden` (compra directa) — alumno, cohorte, oferta, precio snapshot, reservaId,
  estado, timestamps, clave de idempotencia.
- `Subasta` — cohorte, item ofertado, profesor que la lanzó, fecha inicio/fin, puja
  mínima opcional, estado.
- `Puja` — subasta, alumno, monto, reservaId, estado.
- `ItemInventario` — **instancia** poseída por un alumno en una cohorte: origen
  (COMPRA, SUBASTA, DESAFIO), estado (DISPONIBLE, CONSUMIDO, EXPIRADO), fecha de
  consumo. Instancia y no contador, porque RF-REC-05 exige uso único y trazable.

Todas con `cursoCohorteId` obligatorio y baja lógica (RF-NFR-01).

**Lo que NO es suyo:** el saldo de monedas (08), las vidas y el XP (10), la matrícula
(02), los parámetros (12), la identidad (01).

### 5.2 Máquinas de estado que hay que dibujar hoy

- **Orden**: `CREADA → RESERVA_SOLICITADA → RESERVADA → CONFIRMADA` con ramas a
  `RECHAZADA_SALDO`, `FALLIDA_ENTREGA` (→ libera), `EXPIRADA`.
- **Subasta**: `BORRADOR → PROGRAMADA → ABIERTA → EN_CIERRE → ADJUDICADA` con ramas a
  `DESIERTA` (sin pujas) y `CANCELADA` (RF-INT-06).
- **Puja**: `ACTIVA → SUPERADA(liberada) | GANADORA(confirmada) | LIBERADA_POR_CANCELACION`.
- **ItemInventario**: `DISPONIBLE → CONSUMIDO | EXPIRADO | (BLOQUEADO?)`.

El archivo `diagramas-mercado.md` que acompaña este documento trae estas máquinas y
los diagramas de secuencia ya escritos en Mermaid, listos para pegar y ajustar.

### 5.3 Diagramas de la lista mínima para hoy

| Diagrama | Para qué sirve | Quién lo hace |
|---|---|---|
| Contexto (C4 nivel 1/2) del Mercado y vecinos | negociar contratos con 08/10/02/12/11 | Integration lead |
| Clases del dominio | acordar el modelo antes de tocar JPA | Todo el equipo, 45 min |
| Máquinas de estado (4) | evitar estados inventados en código | 2 personas |
| Secuencia: compra directa (feliz + 3 fallos) | es *el* proceso del tema | 2 personas |
| Secuencia: cierre de subasta | el punto de concurrencia | 2 personas |
| ERD del servicio | base para Flyway | 1 persona |

### 5.4 Catálogo de eventos que van a publicar (llevar al Tema 11)

Nombre tentativo, payload mínimo, y si alguien lo espera:

- `mercado.oferta.publicada` → Tema 11 (notificación RF-NOT-02)
- `mercado.compra.confirmada` → Tema 10 (acreditar vida/equipamiento), 12 (analítica)
- `mercado.item.consumido` → Tema 10, 12
- `mercado.subasta.abierta` / `.cerrada` / `.cancelada` → 11, 12
- `mercado.puja.superada` → 11

Y los que **consumen**: `curso.archivado` (02), `curso.activado` (02),
`desafio.recompensa.otorgada` (03/10, si el equipamiento entra por ahí).

Regla del documento de arquitectura: **un evento es un hecho consumado**. Nombres en
pasado, sin pedir respuesta. Si necesitan la respuesta para continuar, es sincrónico
por el gateway, no un evento.

---

## 6. Arquitectura back end

### 6.1 Reglas de plataforma (no se renegocian)

1. El **API Gateway es la única puerta de entrada**.
2. Los servicios se **registran dinámicamente** (service discovery); no hay hosts fijos
   en config.
3. **No hay comunicación directa entre microservicios**: toda llamada sincrónica sale y
   vuelve a entrar por el gateway.
4. **Cada servicio es dueño exclusivo de su base**. Nadie lee la tabla del vecino.
5. Lo asincrónico va por el **bus de eventos**.
6. **Cada entidad tiene un dueño único.**

### 6.2 Patrones que su tema necesita sí o sí

- **Reserva / confirmación / liberación** (una saga corta orquestada por el Mercado).
  Es literalmente el proceso central del tema.
- **Idempotencia**: clave por operación, tabla de claves consumidas. Sin esto, un
  reintento cobra dos veces.
- **Outbox pattern**: si escriben en su base y publican un evento, o son atómicos o
  eventualmente van a tener una compra confirmada sin vida acreditada. La solución
  estándar es guardar el evento en una tabla `outbox` en la misma transacción y
  publicarlo desde ahí.
- **Bloqueo optimista** (`@Version`) en `Subasta` y en `ItemInventario`: el cierre de
  subasta y el consumo de un ítem son carreras reales con 120 sesiones concurrentes.
- **Timeout + reintento con backoff + circuit breaker** (Resilience4j) en las llamadas
  al Banco.
- **Consistencia eventual asumida y visible**: si la vida se acredita por evento, la UI
  tiene que poder mostrar "procesando".

### 6.3 Estructura interna sugerida del servicio

Cada equipo decide su diseño interno. Una arquitectura por capas / hexagonal liviana
alcanza y sobra:

```
com.tup.mercado
├── api/            controllers REST, DTOs, mappers (entrada)
├── application/    casos de uso / services, orquestación de la saga
├── domain/         entidades, value objects, máquinas de estado, reglas puras
└── infrastructure/ repositorios JPA, cliente HTTP al gateway, publisher de eventos
```

Lo importante para el TPI: que las **reglas de negocio sean testeables sin base ni
red**. Ahí es donde después se lucen los tests con JUnit + Mockito.

Patrones de diseño que probablemente aparezcan de forma natural (no los fuercen):
State (máquinas de estado), Strategy (tipos de ítem/efecto), Factory, Repository,
Specification para filtros del catálogo, Observer/publisher para eventos.

---

## 7. Arquitectura front end: las tres propuestas

Contexto que hay que tener claro antes de opinar: son **12 equipos**, un cuatrimestre,
y el producto es una **aplicación web de escritorio** (RF-NFR-05), con un subconjunto
consultivo en móvil (RF-NFR-06). Angular.

La pregunta de fondo no es "cuál es más moderno", sino **en qué momento se integra el
código de los 12 equipos**: en tiempo de desarrollo, en tiempo de build, o en tiempo
de ejecución.

### 7.1 Opción A — Monolito

**Qué es.** Una sola aplicación Angular, un repositorio, un `angular.json`, un build.
Cada tema es un módulo (o un conjunto de rutas con componentes standalone) cargado con
**lazy loading**. La separación es por carpetas y por rutas.

**Cómo se integra.** En tiempo de desarrollo: todos trabajan sobre el mismo árbol de
código.

**Tecnologías/conceptos a dominar:** Angular Router con `loadChildren` /
`loadComponent`, standalone components, signals o RxJS para estado, HttpInterceptor
para el JWT, guards por rol, lazy loading y `preloadingStrategy`.

**Ventajas**
- Setup trivial; el primer día ya tenés algo corriendo.
- Refactor global barato: cambiás una interfaz compartida y el compilador te dice todo
  lo que rompiste.
- Una sola versión de Angular, un solo bundle, cero problemas de dependencias
  compartidas.
- Es lo que la cátedra probablemente espera por defecto.

**Desventajas**
- **Conflictos de merge constantes con 12 equipos** en el mismo repo (rutas, módulo
  raíz, menú, estilos globales).
- Sin fronteras reales: nada impide que el equipo 09 importe un servicio del equipo 03
  y se acoplen sin querer.
- Un build largo para todos; un error de compilación de un equipo frena a los 12.
- El despliegue es todo-o-nada.

**Cuándo es la respuesta correcta:** cuando hay un solo repositorio, una sola entrega,
y alguien con autoridad para ordenar la estructura de carpetas y el code review.

### 7.2 Opción B — Librerías + shell (integración en build time)

**Qué es.** Un **monorepo** (típicamente Nx, que tiene soporte de primera clase para
Angular) donde cada tema es una **librería** Angular independiente
(`libs/mercado/feature-catalogo`, `libs/mercado/domain`, etc.), y una **aplicación
shell** (`apps/shell`) que las consume, define el layout, la navegación y la
autenticación.

Variante: sin monorepo, cada librería se publica como paquete npm versionado y el
shell la instala. Más ceremonia, más aislamiento, más lento para iterar.

**Cómo se integra.** En **build time**: el shell compila e incluye el código de todas
las librerías. Sale un solo bundle (con lazy loading por ruta).

**Tecnologías/conceptos a dominar:** Nx (generators, `nx graph`, `affected`, tags y
**reglas de dependencia** que prohíben que una lib importe a otra), diseño de
librerías Angular (`ng-packagr`), separación *feature / ui / data-access / domain*,
un **design system** compartido como librería, contratos de rutas.

**Ventajas**
- **Fronteras reales y verificables**: Nx puede fallar el build si `mercado` importa
  algo de `desafios`. Eso, con 12 equipos, vale oro.
- Cada equipo tiene su carpeta y sus tests; los merges se concentran en el shell.
- Una sola versión de Angular y de las librerías → cero clase de bugs de MFE.
- `nx affected` corre solo lo que cambió: CI rápido.
- Es el punto medio honesto: aislamiento de código sin complejidad de runtime.

**Desventajas**
- Curva de aprendizaje de Nx y de "cómo se parte una librería" (mucha gente parte mal:
  una lib gigante por tema y no ganó nada).
- El shell es un **cuello de botella organizacional**: alguien tiene que ser su dueño.
- Sigue siendo un único despliegue.

**Cuándo es la respuesta correcta:** exactamente en el escenario de ustedes — muchos
equipos, un producto, una entrega, y necesidad de que nadie pise a nadie.

### 7.3 Opción C — Microfrontends (integración en runtime)

**Qué es.** Cada tema es una **aplicación Angular independiente**, con su repo, su
build y su despliegue. Un **shell/host** las carga **en tiempo de ejecución** y las
monta en una ruta. La tecnología estándar en el ecosistema Angular es **Module
Federation** (Webpack 5) o, con el builder moderno de Angular (esbuild/Vite),
**Native Federation** (`@angular-architects/native-federation`).

**Cómo se integra.** En **runtime**: el host descarga el `remoteEntry` del microfrontend
y lo ejecuta dentro de la misma página.

**Conceptos que hay que entender antes de proponerlo:**
- **Host / remote / remoteEntry**: quién carga a quién y desde qué URL.
- **Shared dependencies y `singleton`**: si dos remotes traen dos copias de Angular o
  de RxJS, se rompe la inyección de dependencias y el router. Hay que compartir
  Angular como singleton y **acordar versiones entre los 12 equipos** — lo que
  paradójicamente vuelve a acoplarlos.
- **Routing anidado**: el host rutea a `/mercado/**` y el remote tiene su propio
  router interno.
- **Estado compartido y comunicación**: entre microfrontends se comunica con eventos
  del navegador (CustomEvent), un bus compartido, o el propio backend. Nunca por
  imports directos.
- **Estilos**: sin encapsulación fuerte (design tokens, prefijos, o Web Components) los
  estilos de un equipo se filtran a los demás.
- **Autenticación**: el token vive en el host y hay que propagarlo (interceptor
  compartido o pasarlo por la API del remote).
- **Versionado y contratos**: si el host cambia la interfaz de montaje, rompe a todos.
- **Infra**: 12 pipelines, 12 despliegues, 12 URLs, CORS, y un ambiente de desarrollo
  donde para probar tu parte necesitás el host corriendo.

**Ventajas**
- **Despliegue independiente real**: el equipo 09 sube una versión sin esperar a nadie.
- Aislamiento fuerte de fallas y de tecnología (en teoría cada uno podría usar otro
  framework — en la práctica acá no aplica, todos usan Angular).
- Refleja la organización (ley de Conway) y es el análogo natural del back end de
  microservicios, lo que lo hace **narrativamente atractivo** en un TP sobre
  microservicios.

**Desventajas**
- **Costo de plataforma alto** para el beneficio real en un cuatrimestre. Requiere que
  alguien sea dueño del shell, del design system y de las versiones compartidas — y
  ese alguien no está asignado en la propuesta (los transversales están *sin dueño*).
- Los bugs son de una clase desagradable: "funciona solo, se rompe integrado".
- Bundle total mayor y peor performance inicial si se comparte mal.
- La ganancia clave (despliegue independiente) **es irrelevante si la entrega es una
  demo por sprint**.

**Cuándo es la respuesta correcta:** cuando los despliegues independientes son un
requerimiento real y hay una plataforma/equipo transversal que sostenga el shell.

### 7.4 Cómo posicionarse en la discusión

En vez de defender una opción por gusto, lleven **criterios de decisión**:

| Criterio | Monolito | Libs + shell | Microfrontends |
|---|---|---|---|
| Costo de arranque | Muy bajo | Medio | Alto |
| Aislamiento entre 12 equipos | Bajo | **Alto (verificable)** | Alto |
| Conflictos de merge | Altos | Bajos | Muy bajos |
| Despliegue independiente | No | No | **Sí** |
| Riesgo de integración tardía | Bajo | Bajo | **Alto** |
| Versión de Angular compartida | Forzada | Forzada | Acordada (frágil) |
| Curva de aprendizaje | Baja | Media (Nx) | Alta (federation) |
| Coherencia con el back end | Baja | Media | Alta |

**Nuestra recomendación argumentable:** **libs + shell en monorepo (opción B)** como
arquitectura de entrega, **con las fronteras dibujadas como si fueran microfrontends**
(cada tema expone un único punto de entrada de rutas, no importa código de otro tema,
consume solo su propia `data-access`). Eso deja la puerta abierta a migrar a Module
Federation más adelante casi sin refactor, y evita pagar hoy el costo de runtime.

Si el grupo se inclina por microfrontends, la condición mínima que deberían exigir:
alguien dueño del shell, versiones de Angular y librerías compartidas congeladas y
escritas, un design system publicado antes del sprint 1, y una prueba de integración
de dos remotes funcionando **en el sprint 0** — no en el 3.

**Aporte concreto que ustedes pueden hacer como equipo 09:** son el único tema con una
funcionalidad **explícitamente habilitada en móvil** (subastas, RF-NFR-06) y otra
**explícitamente no habilitada** (catálogo). Eso los convierte en el caso de prueba
natural de "cómo la plataforma informa que una sección requiere computadora". Es una
propuesta transversal chica, útil y visible.

---

## 8. Scrum y Taiga

### 8.1 Scrum en 15 líneas honestas

- **Roles**: *Product Owner* (ordena el backlog y decide qué vale más), *Scrum Master*
  (facilita, remueve impedimentos, cuida el proceso — no es un jefe), *Developers* (el
  equipo que construye; es auto-organizado).
- **Artefactos**: *Product Backlog* (todo lo que podría hacerse, ordenado), *Sprint
  Backlog* (lo que este sprint se compromete + el plan), *Increment* (lo terminado, que
  cumple la **Definition of Done**).
- **Eventos**: *Sprint* (caja de tiempo fija), *Sprint Planning* (¿qué y cómo?),
  *Daily* (15 min, sincronización del equipo, no reporte al SM), *Sprint Review*
  (mostrar el incremento a los interesados y recoger feedback), *Retrospective*
  (mejorar el proceso), *Refinement* (actividad continua: partir y estimar historias).
- **Compromisos**: el Product Goal, el Sprint Goal y la Definition of Done.

Con 10 personas están en el límite superior de un equipo Scrum. Ver §9.

### 8.2 Definition of Ready y Definition of Done (escríbanlas hoy)

**DoR** — una historia entra al sprint solo si:
- tiene criterios de aceptación en formato verificable;
- cita los RF del PRD que cubre;
- sus dependencias externas están identificadas y, si son bloqueantes, tienen un
  contrato acordado o un mock definido;
- está estimada por el equipo.

**DoD** — propuesta base para el TPI:
- código en la rama principal vía Pull Request con al menos una revisión;
- tests unitarios de la lógica de dominio; test de integración del endpoint;
- endpoint documentado en OpenAPI;
- migración de base versionada (Flyway) si tocó el esquema;
- eventos publicados documentados en el catálogo de eventos;
- sin credenciales ni valores de parámetros hardcodeados;
- demostrable en el ambiente de integración.

### 8.3 Historias de usuario: formato y ejemplos para el mercado

> Como **alumno** de un curso activo, quiero **comprar una vida con mis monedas del
> curso**, para poder seguir intentando desafíos.
> **Criterios de aceptación**
> - Dado saldo ≥ PAR-06 vigente, cuando compro, entonces se descuenta el precio y se
>   acredita 1 vida (RF-INT-01, PAR-06).
> - Dado saldo < precio, entonces la compra se rechaza y no se descuenta nada.
> - Dado que ya tengo el máximo de vidas (PAR-12), entonces la compra no está
>   disponible.
> - Las monedas usadas son del mismo curso (RF-INT-04).
> - La orden guarda el precio con el que se ejecutó (RF-CFG-06).

Buenas historias: **INVEST** (Independent, Negotiable, Valuable, Estimable, Small,
Testable). Las tareas técnicas puras ("crear entidad JPA") son *tasks* dentro de una
historia, no historias.

### 8.4 Taiga en concreto

- Creen el proyecto en modalidad **Scrum** (no Kanban): habilita Backlog, Sprints
  (Taiga los llama *milestones*) y Taskboard.
- Jerarquía: **Epic → User Story → Task**. Una épica por área funcional
  (Catálogo, Compra, Inventario, Subastas, Integraciones), historias adentro, tasks
  para el trabajo real de cada persona.
- **Points**: Taiga permite estimar por rol (UX, Design, Front, Back, Diseño). Usen
  solo los roles que existan de verdad, o una sola escala. Fibonacci (1,2,3,5,8,13) y
  planning poker en el refinement.
- **Statuses**: ajusten el flujo por defecto al de ustedes (ej. New → Ready →
  In progress → Ready for test → Done). No dejen el default si no describe su realidad.
- **Tags**: úsenlos para marcar `dependencia:banco`, `dependencia:roadmap`,
  `spike`, `contrato`, y los `RF-INT-XX`.
- **Issues** para bugs e impedimentos; **Wiki** para el catálogo de eventos, los
  contratos y las decisiones de arquitectura (o linkeen los ADRs del repo).
- **Burndown** del sprint: mírenlo en la daily, no al final.
- Integración con el repo: Taiga soporta webhooks de GitHub/GitLab para mover tarjetas
  con mensajes de commit (`TG-123 #ready-for-test`). Vale la pena configurarlo el
  primer día.

**Sprint 0 (hoy):** carguen en Taiga las **épicas**, las historias del núcleo, los
**spikes** de investigación (§11) y las tareas de setup. Aunque el PO todavía no
priorizó, tener el backlog poblado hace que la primera planning dure una hora en vez
de cuatro.

---

## 9. Cómo repartir 10 personas

Diez personas sobre "el tema más liviano" es el mayor riesgo del equipo: mucha gente
esperando que otro termine. Tres movimientos:

**1. Partirse en células con dueño de vertical.** Cada célula toma una porción end-to-end
(dominio + API + tests + su parte de front), no una capa.

| Célula | Personas | Alcance |
|---|---|---|
| A — Catálogo | 2 | ItemDefinicion, oferta por cohorte, precios desde PAR, consultas |
| B — Compra / Orden | 3 | saga de reserva-confirmación, idempotencia, integración Banco |
| C — Inventario y consumo | 2 | instancias, estados, operación de consumo, integración Roadmap |
| D — Subastas | 2 | máquina de estados, pujas, cierre concurrente (arranca por diseño y mocks) |
| E — Plataforma/integración | 1 (rotativo) | contratos, OpenAPI, docker-compose, CI, ADRs |

**2. Asignar roles transversales** (se suman al trabajo de célula, no lo reemplazan):

- **Scrum Master** (1) — facilita, cuida el tablero, corre las ceremonias.
- **Integration lead** (1) — es la única voz del equipo frente al 08, 10, 02, 11 y 12.
  Sin esto, cinco personas distintas acuerdan cinco contratos distintos.
- **QA/test lead** (1) — DoD, cobertura de la lógica de dominio, datos de prueba.
- **Front lead** (1) — representa al equipo en la discusión de arquitectura de front.
- **Doc/diagram lead** (1) — mantiene diagramas, catálogo de eventos y ADRs vivos.

**3. Tomar carga extra deliberadamente.** El documento lo invita explícitamente: los
extras ("podría ser") son el mecanismo de equilibrio. Candidatos, en orden de valor:
subastas completas, intercambio entre alumnos, catálogo configurable por curso,
vencimiento de ítems, y **ofrecerse por un transversal sin dueño** (alcance móvil es
el que mejor les calza).

Mientras el Banco no exista, **la célula B trabaja contra un stub del Banco** (un
WireMock o un servicio falso en docker-compose que implemente el contrato acordado).
Eso los desbloquea sin esperar a nadie — pero **solo si el contrato está acordado
primero**. De ahí que §3.1 sea la tarea más urgente de la semana.

---

## 10. Ambiente de trabajo

Lo que conviene dejar listo hoy o mañana, aun sin repos definitivos:

**Back end**
- JDK 21 (LTS) + Maven o Gradle; elijan uno y que sea el mismo para todos.
- Spring Boot 3.x: Web, Validation, Data JPA, Actuator.
- PostgreSQL en Docker + **Flyway** para migraciones versionadas.
- **Springdoc-openapi** para generar el contrato desde el código (o escribir el
  OpenAPI a mano primero: *contract-first* es mejor cuando otros equipos consumen).
- Spring Cloud: Gateway, cliente de service discovery (Eureka o Consul, lo define la
  plataforma), Resilience4j.
- Mensajería: RabbitMQ o Kafka — **lo define el Tema 11**, no ustedes. Preparen el
  publisher detrás de una interfaz propia para no atarse.
- Tests: JUnit 5, Mockito, **Testcontainers** (base real en los tests de integración),
  WireMock para simular al Banco.
- Lombok y MapStruct si el equipo los conoce; si no, no los agreguen ahora.

**Front end**
- Node LTS + Angular CLI, versión **acordada entre los 12 equipos**.
- ESLint + Prettier con configuración compartida.
- Un mock de API para no depender del back (`json-server`, MSW, o los mocks de la
  propia herramienta).

**Trabajo en equipo**
- Estrategia de ramas: **trunk-based con ramas cortas** o GitHub Flow. Con sprints de
  1-2 semanas, GitFlow completo es sobredimensionado.
- **Conventional commits** (`feat(mercado): ...`) + referencia a la tarjeta de Taiga.
- Pull Requests con plantilla y al menos una revisión. Es lo que hace que las 10
  personas se enteren de lo que hacen las otras 9.
- **ADRs** (Architecture Decision Records): un `.md` corto por decisión (contexto,
  decisión, consecuencias). Cuando el PO pregunte "¿por qué reserva y no descuento
  directo?", la respuesta ya está escrita.
- Un `README` con: cómo levantar todo, qué variables de entorno, cómo correr los tests.
- `docker-compose.yml` que levante servicio + base + stub del banco.
- CI mínima (GitHub Actions/GitLab CI): build + tests en cada PR. Media hora de setup,
  ahorra semanas.

---

## 11. Qué leer e investigar (spikes para cargar en Taiga)

Prioridad **alta** — antes del sprint 1:

1. **El PRD completo, no solo la Sección 10.** El propio documento avisa que las áreas
   están más entrelazadas de lo que sugiere el índice. Repartan secciones entre las 10
   personas y hagan una puesta en común de 30 minutos.
2. **Scrum Guide** (scrumguides.org) — son 13 páginas, se leen en 20 minutos y evitan
   la mitad de las discusiones sobre proceso.
3. **Documentación de Taiga** (Scrum, backlog, sprints, tags, integración con Git).
4. **Spring Boot + Spring Data JPA**: guías oficiales de spring.io (*Building a RESTful
   Web Service*, *Accessing Data with JPA*).
5. **Spring Cloud Gateway y service discovery**: entender qué hace el gateway y qué
   significa registrarse dinámicamente. La lámina 1 del documento de arquitectura es el
   mapa; la doc oficial es el detalle.
6. **Angular oficial (angular.dev)**: standalone components, router y lazy loading,
   HttpClient e interceptors, signals. Es la doc nueva y es buena.

Prioridad **media** — durante los primeros sprints:

7. **Patrón Saga / reserva-confirmación e idempotencia**: `microservices.io`
   (Chris Richardson) tiene fichas cortas por patrón: Saga, Outbox, API Composition,
   Database per Service. Es la referencia canónica y se lee rápido.
8. **Outbox pattern** — buscar la explicación de Debezium o de microservices.io.
9. **Testcontainers** (docs oficiales) y **WireMock**.
10. **Nx + Angular** (nx.dev): "Angular Monorepo Tutorial", tags y module boundaries.
11. **Module Federation / Native Federation en Angular**:
    `@angular-architects/native-federation` y el material de Manfred Steyer. Léanlo
    aunque terminen eligiendo librerías: es lo que les permite discutir con criterio.
12. **OpenAPI 3** y contract-first; springdoc-openapi.

Prioridad **baja** pero valiosa:

13. **Domain-Driven Design (destilado)** — bounded context, entidad vs value object,
    lenguaje ubicuo. El concepto de *bounded context* explica por qué cada servicio es
    dueño de su base.
14. **Refactoring Guru** para patrones de diseño (State y Strategy son los que más van
    a usar).
15. **Conventional Commits** y **Semantic Versioning** (sitios oficiales, 5 minutos
    cada uno).
16. **Resilience4j** (circuit breaker, retry, timeout).

> Nota: no tengo acceso a búsqueda web en esta conversación, así que verifiquen las
> URLs y las versiones por su cuenta; los nombres de las herramientas y de los
> patrones sí son correctos, pero conviene ir a la fuente.

---

## 12. Agenda propuesta para hoy

Timeboxeada, para 10 personas. Ajusten los tiempos, no el orden.

| Tiempo | Actividad | Salida concreta |
|---|---|---|
| 0:00–0:30 | Puesta en común del PRD: cada uno cuenta la sección que leyó | Entendimiento compartido |
| 0:30–1:00 | Alcance del Tema 09: leer §2 juntos y marcar lo que no se entiende | Lista de dudas |
| 1:00–1:45 | **Mapa de dependencias** (§3): quién necesita qué de quién | Diagrama de contexto + lista de contratos a negociar |
| 1:45–2:00 | Pausa | |
| 2:00–2:45 | **Modelo de dominio** en pizarra: entidades, dueño de cada dato | Diagrama de clases v0 |
| 2:45–3:30 | Máquinas de estado (orden, subasta, puja, ítem) en 2 subgrupos | 4 diagramas |
| 3:30–4:00 | Catálogo de eventos (§5.4) | Lista para llevar al Tema 11 |
| 4:00–4:30 | Roles, células (§9), DoR/DoD (§8.2) | Reparto escrito |
| 4:30–5:00 | Carga en Taiga: épicas, historias del núcleo, spikes | Backlog poblado |
| 5:00–5:30 | Preguntas para el PO (§4) y para la sesión de integración | Documento de 1 página |

Si el día se corta antes, **lo que no puede faltar es**: mapa de dependencias,
preguntas para el PO, y el reparto de roles. El código puede esperar; los contratos no.

---

## 13. Riesgos del equipo (para la retro y para el PO)

| Riesgo | Impacto | Mitigación |
|---|---|---|
| Tema liviano + 10 personas → gente ociosa y solapamiento | Alto | Células con vertical propia (§9) + tomar extras y un transversal |
| Contrato con el Banco no acordado a tiempo | Alto | Acordarlo en la primera sesión de integración; stub mientras tanto |
| Cinco personas negociando cinco contratos distintos | Medio | Un único integration lead |
| Diseñar contadores en vez de historial (XP reducible, ítems consumibles) | Medio | Instancias y eventos, no contadores; está avisado en la lámina 5 |
| Decisiones abiertas del PRD (desmatriculación, caída del sandbox) adoptadas por interpretación propia | Medio | Llevarlas al PO por escrito (§4) |
| Elegir microfrontends sin dueño del shell | Alto (para el proyecto) | Exigir las condiciones mínimas de §7.4 |
| Parámetros hardcodeados (PAR-06/07) | Bajo pero seguro | Está en la DoD |
