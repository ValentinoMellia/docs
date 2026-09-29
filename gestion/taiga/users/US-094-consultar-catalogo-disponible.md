# [G11 — Consultar catálogo disponible]

> **Taiga Ref:** #94 | **ID:** 9535762
> **Épica:** [#90 — G11 — Catálogo Abierto por Plantillas](../epics/EPIC-090-gestion-del-catalogo-de-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** consultar los artículos disponibles en el Mercado de mi curso
*   **Para:** conocer qué artículos puedo adquirir, su tipo y su precio

Notas / Observaciones
---------------------

*   **Reglas de negocio:** el catálogo debe mostrar únicamente los artículos disponibles para el curso/cohorte del ALUMNO. Las monedas utilizadas en el Mercado deben pertenecer al mismo curso donde se realiza el intercambio.
*   **Validaciones:** el ALUMNO debe estar identificado y pertenecer al curso/cohorte consultado. Mientras Identidad y Cursos/Matrícula no estén disponibles, se simula un ALUMNO y un curso/cohorte fijos para poder construir y probar esta historia de punta a punta.
*   **Datos obligatorios:** ID del curso/cohorte, ID del ítem, nombre, tipo, precio, imagen, descripción corta y estado de disponibilidad.
*   **Performance (tiempos, volumen, límites):** la consulta debe devolver el catálogo sin demoras perceptibles para el ALUMNO y soportar múltiples ítems.
*   **Seguridad (roles, permisos, datos sensibles):** un ALUMNO no debe poder consultar ítems pertenecientes a otro curso/cohorte (alineado con RF-USR-07 del PRD: un ALUMNO nunca accede a información fuera de su alcance).
*   **Accesibilidad (WCAG/teclado/lectores):** la información debe poder recorrerse mediante teclado y ser interpretable por lectores de pantalla.
*   **Otros:** las monedas solo pueden utilizarse para adquirir vidas o equipamiento.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** el ALUMNO puede acceder al catálogo correspondiente a su curso/cohorte.
*   **CA2:** cada artículo disponible se muestra como mínimo con nombre, tipo, precio, imagen y descripción corta.
*   **CA3:** el catálogo no muestra artículos pertenecientes a otro curso/cohorte.
*   **CA4:** el ALUMNO puede filtrar el catálogo por tipo de ítem — todos, vidas o equipamiento.
*   **Extras (opcional):** el ALUMNO puede buscar un ítem por nombre mediante un campo de texto libre; si no existen artículos disponibles, se informa que el catálogo se encuentra vacío.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Consulta del catálogo de Mercado

**Escenario 1**

*   **Dado:** un ALUMNO identificado y matriculado en un curso
*   **Cuando:** ingresa al Mercado
*   **Entonces:** el sistema muestra los elementos disponibles para su curso

**Escenario 2**

*   **Dado:** que existe un elemento disponible para el curso del ALUMNO
*   **Cuando:** el ALUMNO consulta el catálogo
*   **Entonces:** el sistema muestra el nombre, tipo, precio, imagen y descripción corta del artículo

**Escenario 3**

*   **Dado:** que existen elementos pertenecientes a otro curso
*   **Cuando:** el ALUMNO consulta su catálogo
*   **Entonces:** el sistema no muestra los elementos del otro curso

**Escenario 4 (extra)**

*   **Dado:** que el ALUMNO está en la pantalla de catálogo con ítems de ambos tipos
*   **Cuando:** selecciona el filtro "Equipamiento"
*   **Entonces:** el sistema muestra únicamente los ítems de tipo equipamiento

Prototipo
---------

*   **Capturas:** front funcionando en Angular (pantalla "Mercado"): tabs Todos/Vidas/Equipamiento, buscador de texto, tarjeta con imagen + nombre + descripción corta + precio + botón Comprar.
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `GET /api/market/catalog?courseCohortId={id}&type={type}&search={text}`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 3
*   **Prioridad (MoSCoW / Numérica):** Must / 1

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 3 | Debe / 1 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado. Identidad y Cursos/Matrícula — mockeados por ahora.
*   **Módulos afectados:** Mercado.
*   **Otros equipos / aprobaciones:** Equipo de Cursos y Matrícula (cuando exista).
*   **Impacto en datos / migraciones:** ninguno significativo; requiere consultar la relación ítem-curso/cohorte.
*   **Riesgos y mitigación (opcional):** riesgo de mostrar información de otro curso. Mitigación: validar siempre el contexto curso/cohorte del ALUMNO, incluso contra el valor simulado.

* * *

