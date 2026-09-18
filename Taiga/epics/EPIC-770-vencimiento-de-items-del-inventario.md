# [G11 — Vencimiento de Ítems del Inventario]

> **Taiga Ref:** #770 | **ID:** 368559
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Mateo Nicolas Presset

## Descripción y Objetivos

## G11 — Vencimiento de Ítems del Inventario

* * *

## Objetivo

Que el PROFESOR pueda ponerles fecha de vencimiento a ciertos ítems de su curso, para generar urgencia de uso, y que los ítems vencidos dejen de estar disponibles para el alumno.

* * *

## Suposiciones y Restricciones

*   Suposiciones:
    *   Es una épica "para más adelante" según la propuesta de arquitectura; no entra en el Sprint 1.
    *   El vencimiento es opcional: un ítem sin fecha de vencimiento no vence nunca.
    *   El PROFESOR define el vencimiento al ofrecer el ítem en su curso (el equipamiento lo define el profesor por curso-cohorte).
    *   El vencimiento puede ser una fecha fija (por ejemplo, el día del parcial) o una cantidad de días desde que el alumno recibe el ítem. *A confirmar con el PO.*
    *   Esta épica cubre el vencimiento por fecha. El cierre de curso ya se trata aparte: al archivar el curso los ítems pasan a solo lectura.
    *   Solo vencen los ítems que no se usaron. Un ítem ya activado sigue sus propias reglas (por ejemplo, la duración del Boost de XP, historia #134).
    *   Las vidas no vencen; se acreditan en Roadmap (G10) y no quedan en el inventario de Mercado.
    *   El campo de vencimiento se agrega al inventario desde la épica de Inventario (#131) aunque no se use todavía, para evitar migraciones después.
*   Restricciones (legales/técnicas):
    *   Si un ítem vence sin usarse, las monedas no se devuelven (el PRD no prevé reembolsos).
    *   Un ítem vencido nunca vuelve a estar disponible y no se informa a Desafíos (G03).
    *   Los ítems solo sirven en el curso donde se obtuvieron (RF-REC-01).
    *   Un cambio de vencimiento hecho por el profesor vale solo para ítems entregados después del cambio (RF-CFG-06).
    *   No se borra nada físicamente; el ítem vencido queda en el historial (RF-NFR-01).

* * *

## Criterios de Aceptación a nivel Épico

*   [ ] El conjunto mínimo de historias permite el flujo: el profesor ofrece un escudo con vencimiento → el alumno lo compra y ve la fecha → recibe un aviso antes de que venza → si no lo usa, vence y deja de aparecer como disponible.
*   [ ] KPI inicial: ningún ítem sigue disponible más de 1 minuto después de su vencimiento.
*   [ ] KPI inicial: 0 ítems vencidos informados a Desafíos.
*   [ ] Sin regresiones críticas en catálogo, compra directa, subastas e inventario.
*   [ ] Observabilidad y alertas configuradas: registro de cada ítem vencido (alumno, curso, ítem, fecha).
*   [ ] Documentación de uso y operación publicada: reglas de vencimiento y evento de ítem vencido documentados.

* * *

## Dependencias / Impactos

*   Servicios / APIs: Mercado (dueño); Motor de Desafíos G03 (deja de recibir ítems vencidos); Notificaciones (aviso previo y aviso de vencimiento); Backoffice G12 (reportes de ítems vencidos, opcional).
*   Módulos afectados: Mercado — Catálogo, Inventario.
*   Otros equipos: equipo de Notificaciones (formato de los avisos); PO (tipo de vencimiento: fecha fija, días desde la entrega, o ambos).
*   Impacto en datos / migraciones: campo de vencimiento en la oferta del catálogo; `expiration_datetime` en el inventario; estado "vencido" en el inventario.
*   Feature toggles / flags: sí, uno para activar el vencimiento por curso. Se retira cuando la funcionalidad esté estable.

## Historias de Usuario Asociadas (5)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#778** | G11 — Configurar el vencimiento de un ítem del curso | New | — | [US-778-configurar-el-vencimiento-de-un-item-del-curso.md](../users/US-778-configurar-el-vencimiento-de-un-item-del-curso.md) |
| **#785** | G11 — Ver cuándo vencen mis ítems | New | — | [US-785-ver-cuando-vencen-mis-items.md](../users/US-785-ver-cuando-vencen-mis-items.md) |
| **#786** | G11 — Recibir un aviso antes de que venza un ítem | New | — | [US-786-recibir-un-aviso-antes-de-que-venza-un-item.md](../users/US-786-recibir-un-aviso-antes-de-que-venza-un-item.md) |
| **#787** | G11 — Vencer automáticamente los ítems no usados | New | — | [US-787-vencer-automaticamente-los-items-no-usados.md](../users/US-787-vencer-automaticamente-los-items-no-usados.md) |
| **#788** | G11 — Consultar los ítems vencidos del curso | New | — | [US-788-consultar-los-items-vencidos-del-curso.md](../users/US-788-consultar-los-items-vencidos-del-curso.md) |
