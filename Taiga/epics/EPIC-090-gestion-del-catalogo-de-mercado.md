# [G11 — Catálogo Abierto por Plantillas]

> **Taiga Ref:** #90 | **ID:** 367203
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Melina Yain Medina

## Descripción y Objetivos

### Objetivo

Que el PROFESOR arme el catálogo de su curso-cohorte eligiendo una plantilla base que ofrece Mercado y configurando sus parámetros, y que el ALUMNO vea únicamente las ofertas activas de su curso con su precio, su efecto y su disponibilidad.

### Suposiciones y Restricciones

*   **Suposiciones:**
    *   El catálogo es **ABIERTO**: Mercado ofrece plantillas base y el PROFESOR las configura para su cohorte. Mercado no crea ítems libres.
    *   Plantillas base: `SHIELD`, `BOOST_XP`, `BOOST_COINS` y `LIFE`.
    *   Parámetros configurables por el PROFESOR: `coinPrice`, `stock` opcional, y los propios del tipo — `charges` y `applicableChallenges` (SHIELD); `multiplier`, `mode` (`TTL` / `PER_EXAM`), `durationMinutes`, `attempts`, `consumptionRule` (BOOST_*); `livesGranted` (LIFE).
    *   El **inventario del alumno NO es de Mercado**: las instancias compradas las persiste **Banco**. El catálogo solo define qué se ofrece.
    *   Mientras Usuarios y Cursos no estén disponibles, se simulan el profesor, el alumno y el curso-cohorte.
*   **Restricciones (legales/técnicas):**
    *   Las recompensas y las monedas son por curso (RF-REC-01, RF-INT-04).
    *   Baja lógica en todas las entidades, sin borrado físico (RF-NFR-01).
    *   La orden guarda el precio con el que se ejecutó (RF-CFG-06): un cambio de precio rige solo hacia adelante.
    *   Una oferta inactiva no se puede comprar, aunque el alumno tenga la vitrina abierta desactualizada.
    *   El PROFESOR solo publica en los cursos que dicta.
    *   **Pendiente con el PO:** si el precio libre del profesor convive con algún límite global de Administración (RF-CFG-04/05, PAR-06/07).

### Criterios de Aceptación a nivel Épico

*   Flujo extremo a extremo: **el PROFESOR elige una plantilla → la configura y la publica en su curso → el ALUMNO la ve en la vitrina con precio, efecto y stock restante**.
*   KPI inicial: cero ofertas de otro curso-cohorte visibles en las pruebas de aislamiento.
*   KPI inicial: cero ofertas publicadas con una configuración inválida para su tipo de plantilla.
*   Sin regresiones críticas en la compra directa, que consume este catálogo.
*   Observabilidad: log de altas, ediciones y cambios de estado de cada oferta, con autor y fecha.
*   Documentación publicada: contrato OpenAPI de plantillas y catálogo, disponible para el front y para el resto de los equipos.

### Dependencias / Impactos

*   **Servicios / APIs:** Mercado (dueño). Usuarios (roles `ROLE_PROFESSOR` / `ROLE_STUDENT`) y Cursos (curso-cohorte y matrícula) — simulados mientras esos equipos no tengan nada disponible.
*   **Módulos afectados:** Mercado — Catálogo. Lo consume la épica de Compra Directa.
*   **Otros equipos:** Cursos (pertenencia al curso), Usuarios (roles), Administración (posibles límites globales de precio, a confirmar).
*   **Impacto en datos / migraciones:** tablas `item_base_template` y `course_catalog_offer` con la configuración por tipo de plantilla, más `stock` y `available_stock`.
*   **Feature toggles / flags:** no se requieren inicialmente.

> Reemplaza el modelo anterior de catálogo administrado por ADMIN con ítems concretos y stock fijo obligatorio. Ver `CONTEXTO-MERCADO-SPRINT1.md` §5 y §12-E: el stock ahora es un campo opcional por oferta (vacío = ilimitado, > 0 = tope finito), no una prohibición absoluta.

## Historias de Usuario Asociadas (7)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#92** | G11 — Publicar una oferta en mi curso a partir de una plantilla | New | 5 | [US-092-crear-un-item.md](../users/US-092-crear-un-item.md) |
| **#94** | G11 — Consultar la vitrina de mi curso | New | 3 | [US-094-consultar-catalogo-disponible.md](../users/US-094-consultar-catalogo-disponible.md) |
| **#95** | G11 — Editar una oferta publicada | New | 3 | [US-095-modificar-un-item.md](../users/US-095-modificar-un-item.md) |
| **#96** | G11 — Activar o desactivar una oferta | New | 3 | [US-096-activar-o-desactivar-un-item.md](../users/US-096-activar-o-desactivar-un-item.md) |
| **#98** | G11 — Consultar el detalle de una oferta | New | 2 | [US-098-consultar-detalle-de-un-articulo.md](../users/US-098-consultar-detalle-de-un-articulo.md) |
| **#945** | G11 — Listar las plantillas base disponibles | New | 2 | *(creada en Taiga el 17/09/2026, sin archivo local — pendiente de exportar)* |
| **#946** | G11 — Ver el catálogo completo de mi curso | New | 2 | *(creada en Taiga el 17/09/2026, sin archivo local — pendiente de exportar)* |
