# [G11 — Gestión del Catálogo de Mercado]

> **Taiga Ref:** #90 | **ID:** 367203
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Melina Yain Medina

## Descripción y Objetivos

Objetivo
--------

Permitir la gestión y consulta de los ítems disponibles en el Mercado, asegurando que los alumnos puedan visualizar los productos habilitados para su curso/cohorte y conocer sus características y precios antes de realizar una compra.

Suposiciones y Restricciones
----------------------------

*   **Suposiciones:**
    *   Los ítems de Mercado están asociados al contexto del curso/cohorte correspondiente.
    *   Los ítems pueden ser de los tipos contemplados por el Mercado: **vidas** y **equipamiento** (RF-DES-04 los llama BASICO/MEDIO/AVANZADO a nivel desafío, pero la clasificación propia del Mercado es vida/equipamiento).
    *   El rol **ADMIN** es quien administra el catálogo global, en línea con RF-CFG-04 del PRD (la economía de gamificación es configuración global administrada exclusivamente por ADMIN).
    *   El catálogo será la fuente de información utilizada por el rol **ALUMNO** para conocer qué ítems se encuentran disponibles para la compra.
    *   Mientras los servicios de Identidad, Cursos/Matrícula y Banco no estén disponibles (equipos en desarrollo inicial), se trabaja con un ALUMNO y un curso/cohorte simulados de forma fija, y con el precio persistido directamente en el ítem, para no bloquear el desarrollo de esta épica.
*   **Restricciones (legales/técnicas):**
    *   Las monedas son propias del curso y no pueden utilizarse fuera de dicho contexto (RF-INT-04).
    *   El Mercado no administra directamente el saldo de monedas del ALUMNO; esa gestión corresponde al módulo Banco.
    *   Solo deben visualizarse ítems correspondientes al curso/cohorte del ALUMNO.
    *   Los ítems no disponibles o no publicados no deben aparecer como opciones de compra.
    *   El PROFESOR no puede sobreescribir los parámetros de economía definidos por ADMIN (RF-CFG-05); en esta primera etapa, el PROFESOR no interviene en el catálogo — la curaduría por cohorte queda para una iteración posterior.

Criterios de Aceptación a nivel Épico
-------------------------------------

*   El conjunto mínimo de historias permite el flujo extremo a extremo **ADMIN da de alta un ítem → el ALUMNO lo consulta en el catálogo de su curso/cohorte con su precio, tipo e imagen vigentes**.
*   KPIs iniciales alcanzan: cero ítems visibles fuera del curso/cohorte del ALUMNO en las pruebas de aislamiento; el catálogo responde sin demoras perceptibles para el usuario con el volumen de ítems del MVP.
*   Sin regresiones críticas en el flujo de catálogo ni en el resto de Mercado.
*   Observabilidad y alertas configuradas: log de altas, modificaciones y cambios de estado de ítems, con autor y fecha.
*   Documentación de uso y operación publicada: contrato de los endpoints de catálogo disponible para su consumo posterior por el flujo de compra.

Dependencias / Impactos
-----------------------

*   **Servicios / APIs:** Servicio de Mercado (dueño). Servicio de Identidad y Usuarios, Servicio de Cursos y Matrícula y Servicio Banco — **mockeados mientras esos equipos no tengan nada disponible**.
*   **Módulos afectados:** Mercado.
*   **Otros equipos:** Equipo responsable de Identidad, Equipo responsable de Cursos y Matrícula, Equipo responsable de Banco — a coordinar cuando corresponda.
*   **Impacto en datos / migraciones:** creación de la estructura para almacenar los ítems del catálogo (nombre, tipo, precio, imagen, descripción corta, estado y curso/cohorte asociado como atributo directo del ítem).
*   **Feature toggles / flags:** no se requieren inicialmente. En caso de habilitar funcionalidades de manera progresiva (por ejemplo, al conectar el precio real desde Banco), se podrá utilizar una marca de función para controlar su disponibilidad. Plan de retiro: se elimina cuando la integración real esté estable.

## Historias de Usuario Asociadas (5)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#92** | G11 — Crear un ítem | New | — | [US-092-crear-un-item.md](../users/US-092-crear-un-item.md) |
| **#94** | G11 — Consultar catálogo disponible | New | — | [US-094-consultar-catalogo-disponible.md](../users/US-094-consultar-catalogo-disponible.md) |
| **#95** | G11 — Modificar un ítem | New | — | [US-095-modificar-un-item.md](../users/US-095-modificar-un-item.md) |
| **#96** | G11 — Activar o desactivar un ítem | New | — | [US-096-activar-o-desactivar-un-item.md](../users/US-096-activar-o-desactivar-un-item.md) |
| **#98** | G11 — Consultar detalle de un artículo | New | — | [US-098-consultar-detalle-de-un-articulo.md](../users/US-098-consultar-detalle-de-un-articulo.md) |
