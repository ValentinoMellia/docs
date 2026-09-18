# [G11 — Crear un ítem]

> **Taiga Ref:** #92 | **ID:** 9535755
> **Épica:** [#90 — G11 — Gestión del Catálogo de Mercado](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ADMIN
*   **Quiero:** crear un nuevo artículo para el catálogo
*   **Para:** incorporar nuevas vidas o equipamientos disponibles en el Mercado

Notas / Observaciones
---------------------

*   **Reglas de negocio:** solo se pueden incorporar al Mercado las categorías definidas — vidas y equipamiento. Las monedas son el medio de intercambio y no hay un artículo que pueda adquirirse como tal. La configuración de la economía del Mercado es potestad exclusiva de ADMIN (RF-CFG-04); ni PROFESOR ni ALUMNO pueden dar de alta ítems.
*   **Validaciones:** nombre, tipo, precio, imagen y descripción corta deben estar informados. El precio debe ser válido según las reglas del Mercado.
*   **Datos obligatorios:** nombre, tipo de artículo (vida/equipamiento), precio, imagen (URL), descripción corta, curso/cohorte y estado.
*   **Performance (tiempos, volumen, límites):** la creación debe procesarse como una operación transaccional.
*   **Seguridad (roles, permisos, datos sensibles):** solo ADMIN puede crear ítems. Todo intento de un rol distinto se rechaza y queda registrado.
*   **Accesibilidad (WCAG/teclado/lectores):** los formularios deben ser navegables mediante teclado y contar con etiquetas accesibles.
*   **Otros:** el artículo creado debe quedar asociado al contexto correspondiente (curso/cohorte simulado mientras Cursos y Matrícula no exista).

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** ADMIN puede crear un ítem ingresando todos los datos obligatorios.
*   **CA2:** el sistema rechaza la creación si falta algún dato obligatorio.
*   **CA3:** el elemento creado queda asociado al curso/cohorte indicado.
*   **Extras (opcional):** al finalizar la creación, el sistema informa que el artículo fue creado correctamente.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Creación de artículos del Mercado

**Escenario 1**

*   **Dado:** un ADMIN autenticado
*   **Cuando:** completa correctamente los datos obligatorios y confirma
*   **Entonces:** el sistema crea el ítem y lo asocia al curso/cohorte correspondiente

**Escenario 2**

*   **Dado:** un ADMIN autenticado
*   **Cuando:** intenta crear un artículo sin informar el precio
*   **Entonces:** el sistema rechaza la operación e indica el dato faltante

**Escenario 3**

*   **Dado:** un ALUMNO o PROFESOR autenticado (sin permisos de ADMIN)
*   **Cuando:** intenta crear un elemento
*   **Entonces:** el sistema rechaza la operación por falta de permisos

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `POST /api/market/items`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño). Identidad — solo para validar el rol ADMIN del usuario que crea; mockeable con un rol fijo mientras tanto.
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** ninguna bloqueante para esta historia.
*   **Impacto en datos / migraciones:** requiere persistencia de nuevos registros de ítems.
*   **Riesgos y mitigación (opcional):** evitar creación de artículos con categorías no contempladas por el Mercado; validar el rol ADMIN en cada intento de alta.

