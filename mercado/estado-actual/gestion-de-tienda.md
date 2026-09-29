# Gestión de tienda (ofertas creadas por profesores): estado actual

> **Estado al 29/09/2026** · `tpi-market` `develop` @ `7528610`. Verificado leyendo el código; la app no se ejecutó.
> Diseño funcional del catálogo abierto: [`../catalogos/README.md`](../catalogos/README.md) (marcado como parcialmente desactualizado; este documento prevalece en lo implementado).

## 1. En una frase

Un profesor publica en su curso **ofertas** basadas en una de cuatro **plantillas base** (escudo, boost de XP, boost de monedas, vida), fijando precio, stock, vigencia de la publicación y parámetros del ítem; puede editarlas, activarlas y desactivarlas. Los alumnos ven solo las ofertas activas y vigentes. No hay eventos ni llamadas salientes al publicar: la oferta viaja a Accounting recién al comprarse.

| Aspecto | Estado |
|---|---|
| Plantillas base + siembra | ✅ |
| Publicar / editar oferta | ✅ (con un bug de stock, §4) |
| Activar / desactivar (3 caminos) | ✅ |
| Listado del profesor y vitrina del alumno | ✅ |
| Vencimiento de la publicación (US-2402) | ✅ |
| Caducidad del **ítem** (US-785) | ⛔ Descartada; quedan restos en el código |
| Verificación de que el profesor es docente del curso | ⚠️ Mock |
| Baja lógica (`DELETE`), cohortes, edición de plantilla/config | ❌ No existen |

## 2. API (prefijo `/api/market`)

| Método y ruta | Roles | Qué hace |
|---|---|---|
| `GET /templates` | PROFESSOR, ADMIN, GESTOR | Plantillas activas con metadata de parámetros (`type`, `required`, `minValue`, `allowedValues`, `appliesWhen`) para construir el formulario |
| `GET /courses/{courseId}/catalog/manage?itemType=` | PROFESSOR, ADMIN, GESTOR (+ chequeo de docente) | Lista **todas** las ofertas del curso, activas o no, vencidas o no, con `unitsSold` y `itemValidityDays` |
| `POST /courses/{courseId}/catalog/manage` | ídem | Publica una oferta → `201` |
| `PATCH /courses/{courseId}/catalog/manage/{offerId}` | ídem | Edita nombre, descripción, precio, stock, `active` |
| `PATCH /courses/{courseId}/catalog/manage/offers/{itemId}/status` | PROFESSOR, ADMIN, GESTOR, MS (los roles administrativos saltan el chequeo de docente) | Activa/desactiva por `{active}` o `{status: ACTIVE\|INACTIVE}` |
| `PATCH /offers/{id}/status` (y `/api/v1/market/offers/{id}/status`) | ADMIN, GESTOR, MS | Activa/desactiva a nivel global |
| `GET /offers/{id}` | STUDENT, PROFESSOR, ADMIN, GESTOR, MS | Detalle (US-098) |

El Swagger (`docs/api_doc/swagger.json`, regenerado el 27/09) coincide con estas rutas.

**Cuerpo de publicación** (`CatalogOfferPublishDto`): `templateId`(≤50), `itemType`, `customName`(≤150), `customDescription`(≤500), `coinPrice`(>0), `totalStock` (>0, `null` = ilimitado), `publicationTtlMinutes` (>0, `null` = sin vencimiento), `itemValidityDays` (residual), y `configuration` con `charges`, `applicableChallenges`, `multiplier`, `boostMode`, `durationMinutes`, `attempts`, `consumptionRule`, `livesGranted`.
**Errores:** `400 validation-error` (con mensaje por campo "Este campo es requerido para el tipo de item seleccionado"), `400 offer-update-not-allowed`, `403 professor-not-assigned`, `404 catalog-offer-not-found`.

## 3. Modelo

**Tablas** (JPA; sin migraciones versionadas): `course_catalog_offers` (oferta por curso, índices `(course_id, active, deleted)` y `item_type`) e `item_base_templates`.

**Plantillas sembradas al arrancar** (`base-templates.json`, idempotente):

| `templateId` | Tipo | Precio base | Configuración por defecto |
|---|---|---|---|
| `tpl-shield-base` | `SHIELD` | 350 | 2 cargas, `PRACTICAL_ONLY` |
| `tpl-boost-xp` | `BOOST_XP` | 400 | ×1,50, `TTL` 120 min |
| `tpl-boost-coins` | `BOOST_COINS` | 300 | ×2,00, `PER_EXAM`, 3 intentos, `CONSUME_ON_PASS_ONLY` |
| `tpl-life-potion` | `LIFE` | 500 | 1 vida |

En dev/docker se siembran además 6 ofertas de ejemplo para `COURSE_PROG4_2026` y `COURSE_OTHER_9999`.

**Parámetros obligatorios por tipo** (`ValidOfferConfigurationValidator`): `SHIELD` → `charges`, `applicableChallenges`; `BOOST_XP`/`BOOST_COINS` → `multiplier`, `boostMode`, y `durationMinutes` si `TTL` o `attempts` + `consumptionRule` si `PER_EXAM`; `LIFE` → `livesGranted` (y no admite `itemValidityDays`).

**Enums:** `ItemType` {SHIELD, BOOST_XP, BOOST_COINS, LIFE}, `BoostMode` {TTL, PER_EXAM}, `ChallengeApplicability` {ALL, THEORETICAL_ONLY, PRACTICAL_ONLY, NO_EXAMS}, `ConsumptionRule` {ALWAYS_CONSUME, CONSUME_ON_PASS_ONLY}.

## 4. Reglas y comportamientos

- **Publicar:** `availableStock = totalStock`, `active = true`, `unitsSold = 0`, `publicationExpiresAt = ahora + publicationTtlMinutes` (o sin vencimiento).
- **Vencimiento de la publicación (US-2402).** No cambia el flag `active`: la vitrina oculta la oferta cuando `publicationExpiresAt ≤ ahora` (comparación estricta) pero el profesor la sigue viendo; **reactivar** una oferta vencida por edición falla con `400`; comprarla falla con `409 catalog-offer-expired`. El TTL se fija **solo al publicar** y se evalúa de forma perezosa por consulta y reloj (no hay job).
- **Activar/desactivar:** hay tres caminos (por curso, global/admin y por el `PATCH` de edición).
- **Edición de stock:** `totalStock` no puede quedar por debajo de `unitsSold`; `unlimitedStock = true` deja ambos campos en `null`; `totalStock` y `unlimitedStock` son excluyentes.
- **Inmutabilidad:** tras publicar no se puede cambiar plantilla ni configuración (el DTO de edición no los incluye).
- **Ítem vencido (US-785) descartado.** Se fusionó en el PR #44 y se revirtió en el #60 (issue #56). Quedan restos: la columna `item_validity_days`, el campo en los DTO de publicación y gestión, y el validador. **No** se envía a Accounting (`toPayload` lo omite, con test de regresión) ni se muestra en la vitrina. El cambio `openspec/changes/item-validity-duration` está desactualizado y contradice `develop`.

### Defectos conocidos

1. **`units_sold` nunca se incrementa.** Las compras solo descuentan `available_stock`; confirmar una orden no toca `units_sold`. Consecuencias: el piso "no bajar el stock por debajo de lo vendido" no se dispara nunca; **editar `totalStock` después de vender deja `availableStock = nuevoTotal`**, o sea devuelve al stock las unidades vendidas y reservadas (sobreventa); y `unitsSold` siempre vale `0` en el listado del profesor. Los tests solo lo cubren con fixtures cargados a mano.
2. **Autorización débil en el servicio.** `validateProfessorAccess` y `validateAdminRole` comparan con `contains()` sobre el header de roles (por ejemplo `"MS"`), **se saltean** el chequeo de rol si el header viene vacío y el de asignación docente si falta `userId`. La barrera real es `@PreAuthorize`. Además la asignación docente es un mock, y ADMIN/GESTOR pasan por ese mock al publicar y editar (solo los endpoints de estado tienen bypass).
3. **Publicar no valida la plantilla.** No verifica que `templateId` exista ni que coincida con `itemType`, no exige el multiplicador mínimo 1,10 de la plantilla (solo `> 0`) ni ata el precio al de la plantilla.
4. **El detalle no chequea vencimiento.** `GET /courses/{courseId}/catalog/{itemId}` responde `200` a un alumno para una oferta vencida pero activa (que la lista oculta); recién la compra falla con `409`.
5. **Sin "borrar":** la columna `deleted` no tiene escritor; no hay endpoint de baja lógica.
6. **Sin cohortes:** una oferta se liga a un `courseId` opaco (el "curso-cohorte"); no hay entidad de cohorte.

## 5. Integraciones

Solo `CourseInstructorClient` (**mock**). Publicar, editar y desactivar no emiten eventos ni llaman a otros servicios; el snapshot de la oferta va a Accounting únicamente al comprar (ver [compra](./compra.md)). Consecuencia a tener en cuenta: Accounting **no conoce** ofertas, precios, vigencias ni el catálogo del profesor.

## 6. Tests

`CourseCatalogManageServiceTest` (29), `CourseCatalogManageControllerTest` (22), `CatalogOfferControllerTest` (12), `StorefrontCatalogServiceTest` (14), `ValidOfferConfigurationValidatorTest` (17) y aceptación de plantillas (10), activación (7) y vencimiento de publicación (7: bordes del TTL, vitrina, gestión, reactivación).
**Huecos:** deriva de `unitsSold` de punta a punta, publicar con plantilla inexistente, visibilidad del detalle de una oferta vencida.

## 7. Historias de usuario (según código, no según Taiga)

| US | Estado | Evidencia |
|---|---|---|
| US-092 publicar oferta | ✅ | PR #35 |
| US-094 vitrina del alumno | ✅ | PR #2 |
| US-095 editar oferta | ✅ con bug de stock (§4.1) | PR #35 |
| US-096 activar/desactivar | ✅ (3 endpoints) | PR #45, #62 |
| US-098 detalle de oferta | ✅ (sin chequeo de vencimiento) | PR #45 |
| US-945 plantillas base | ✅ | PR #37, #43 |
| US-946 listado del profesor | ✅ | PR #1 |
| US-2402 vencimiento de publicación | ✅ | PR #39, #51, #52 |
| US-785 caducidad del ítem en la vitrina | ⛔ Fusionada (PR #44) y revertida (PR #60, issue #56) | PR #44, #60 |
| US-778, US-786–788 (vencimiento y aviso de ítems) | Sin evidencia en este repo (no verificado en Taiga); el ítem no tiene fecha de vencimiento ni en Mercado ni en Accounting | — |
| US-093, US-097 | Sin evidencia en commits, PRs ni issues (no verificado en Taiga) | — |
| US-130–137 (inventario, boost, escudo, regalos) | Sin evidencia en este repo; corresponden a Accounting/otros servicios | — |
