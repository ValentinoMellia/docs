# [G11 — Publicar una oferta en mi curso a partir de una plantilla]

> **Taiga Ref:** #92 | **ID:** 9535755
> **Épica:** [#90 — G11 — Catálogo Abierto por Plantillas](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** 5
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

### Descripción (Como / Quiero / Para)

*   **Como:** PROFESOR
*   **Quiero:** publicar una oferta en el catálogo de mi curso eligiendo una plantilla y configurando sus parámetros
*   **Para:** ofrecer a mis alumnos ítems pensados para mi cursada

### Notas / Observaciones

*   **Reglas de negocio:** Mercado ofrece 4 plantillas base (`SHIELD`, `BOOST_XP`, `BOOST_COINS`, `LIFE`). El PROFESOR no crea ítems desde cero: elige una plantilla y la configura. Cada oferta pertenece a un único curso-cohorte.
*   **Validaciones:** `coinPrice` ≥ 1. `stock` vacío (ilimitado) o mayor a 0. Según el tipo: `SHIELD` → `charges` ≥ 1 y `applicableChallenges`; `BOOST_*` → `multiplier` y, según `mode`, `durationMinutes` (TTL) o `attempts` + `consumptionRule` (PER_EXAM); `LIFE` → `livesGranted` ≥ 1.
*   **Datos obligatorios:** plantilla, curso-cohorte, nombre, descripción corta, precio en monedas, configuración propia del tipo y estado.
*   **Performance (tiempos, volumen, límites):** alta transaccional; volumen bajo (decenas de ofertas por curso).
*   **Seguridad (roles, permisos, datos sensibles):** solo `ROLE_PROFESSOR`, y solo en los cursos que dicta. Cualquier otro intento se rechaza y queda registrado.
*   **Accesibilidad (WCAG/teclado/lectores):** formulario navegable por teclado, con cada error de validación asociado a su campo.
*   **Otros:** el precio queda guardado en la oferta; además, cada orden guarda su propio precio (RF-CFG-06). Mercado no crea inventario: las instancias compradas las persiste Banco.

### Criterios de Aceptación (CA)

*   **CA1:** el PROFESOR publica una oferta eligiendo una plantilla y completando los datos obligatorios, y esa oferta queda visible en la vitrina del curso.
*   **CA2:** si falta un dato obligatorio o la configuración no corresponde al tipo de plantilla, la publicación se rechaza indicando el campo con problema.
*   **CA3:** un rol distinto de PROFESOR, o un profesor en un curso que no dicta, recibe un rechazo por permisos.
*   **Extras (opcional):** al publicar, el sistema confirma el alta y muestra la oferta con su stock inicial.

### BDD (mínimo 3 escenarios)

**Característica:** Publicación de ofertas del catálogo a partir de plantillas base

**Escenario 1**

*   **Dado:** un PROFESOR autenticado en un curso que dicta
*   **Cuando:** elige la plantilla de escudo, define precio y cargas, y publica
*   **Entonces:** la oferta queda activa en el catálogo de ese curso

**Escenario 2**

*   **Dado:** un PROFESOR que elige una plantilla de boost por tiempo
*   **Cuando:** publica sin indicar la duración
*   **Entonces:** el sistema rechaza la publicación e indica que falta la duración

**Escenario 3**

*   **Dado:** un ALUMNO autenticado
*   **Cuando:** intenta publicar una oferta
*   **Entonces:** el sistema rechaza la operación por falta de permisos

### Prototipo

*   **Mock API / Swagger:** `POST /api/v1/market/courses/{courseId}/catalog/items`

### Estimación / Prioridad

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Must / 1

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Must / 1 |

### Dependencias / Impactos

*   **Servicios involucrados:** Mercado (dueño). Usuarios (rol del profesor) y Cursos (curso-cohorte) — simulados mientras no estén disponibles.
*   **Módulos afectados:** Mercado — Catálogo.
*   **Otros equipos / aprobaciones:** Administración — pendiente confirmar si el precio libre convive con límites globales (RF-CFG-04/05).
*   **Impacto en datos / migraciones:** tabla de ofertas del curso con la configuración por tipo de plantilla.
*   **Riesgos y mitigación:** publicar configuraciones inválidas para el tipo elegido; se mitiga validando por tipo en el backend y cubriéndolo con pruebas.

> Reemplaza a la historia anterior "Crear un ítem" (catálogo administrado por ADMIN), que quedó sin efecto con el catálogo abierto por plantillas.
