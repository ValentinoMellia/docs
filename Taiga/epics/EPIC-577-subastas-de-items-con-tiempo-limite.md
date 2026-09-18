# [G11 — Subastas de Ítems con Tiempo Límite]

> **Taiga Ref:** #577 | **ID:** 368163
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Mateo Nicolas Presset

## Descripción y Objetivos

## G11 — Subastas de Ítems con Tiempo Límite

* * *

## Objetivo

Que el PROFESOR pueda subastar un ítem de equipamiento durante un tiempo definido y que los alumnos del curso compitan ofertando monedas. Al terminar, quien ofertó más paga y recibe el ítem, y nadie pierde monedas sin recibir nada.

* * *

## Suposiciones y Restricciones

*   Suposiciones:
    *   El alumno puede seguir las subastas y ofertar también desde el celular (RF-NFR-06).
    *   Cada participante tiene retenida su oferta completa hasta que la subasta termina (RF-INT-05). Retener solo a quien va ganando queda para más adelante y necesita aprobación del PO.
    *   **Solo se subasta equipamiento** (escudos y boosts). Las vidas no, porque el tope de 3 vidas (PAR-12) puede cambiar durante los días que dura una subasta. *A confirmar con el PO.*
    *   Si el curso se archiva con subastas abiertas, esas subastas se cancelan y se devuelven todas las monedas. *A confirmar con G02 y el PO.*
    *   Mientras el Banco no tenga lista la retención por varios días, Mercado usa un Banco simulado para poder avanzar.
    *   Quedan fuera de esta épica: extender la subasta si alguien oferta en el último minuto, subastar vidas e intercambio entre alumnos.
*   Restricciones (legales/técnicas):
    *   Las monedas usadas tienen que ser del mismo curso de la subasta (RF-INT-04).
    *   Una oferta solo puede subir; nunca bajar ni retirarse mientras la subasta está abierta (RF-INT-05).
    *   Las monedas retenidas no se pueden usar para comprar en el catálogo ni en otra subasta.
    *   Si el profesor cancela, nadie recibe el ítem y se devuelven todas las monedas (RF-INT-06).
    *   El cierre lo decide Mercado. El Banco no puede liberar una retención antes de que la subasta termine.
    *   Cada subasta se cierra una sola vez, aunque Mercado esté corriendo en más de una instancia.
    *   No se borra nada físicamente; solo baja lógica (RF-NFR-01).

* * *

## Criterios de Aceptación a nivel Épico

*   [ ] El conjunto mínimo de historias permite el flujo: el profesor lanza una subasta → dos alumnos ofertan y uno mejora su oferta → la subasta termina → el ganador paga y recibe el ítem → al otro se le devuelven las monedas. También funcionan los casos de subasta sin ofertas y subasta cancelada.
*   [ ] KPI inicial: 0 subastas entregadas dos veces en la prueba con dos instancias de Mercado.
*   [ ] KPI inicial: 0 monedas que queden retenidas al terminar las pruebas de fallas (Banco caído o Mercado reiniciado durante el cierre).
*   [ ] KPI inicial: cierre completo en menos de 2 minutos con 50 ofertas.
*   [ ] KPI inicial: un cambio de líder se ve en pantalla en menos de 2 segundos.
*   [ ] Sin regresiones críticas en catálogo, compra directa e inventario.
*   [ ] Observabilidad y alertas configuradas: registro de cada subasta de punta a punta, alerta si un cierre tarda más de 2 minutos y alerta si una devolución lleva más de 15 minutos sin confirmarse.
*   [ ] Documentación de uso y operación publicada: endpoints en Swagger, diagrama de estados de la subasta y contratos de eventos.

* * *

## Dependencias / Impactos

*   Servicios / APIs: Mercado (dueño); Banco G08 (retener, ampliar, cobrar y devolver monedas); Cursos y Matrícula G02 (matrícula, curso archivado, baja de alumno); Identidad G01 (roles); Notificaciones (avisos); Backoffice G12 (reportes).
*   Módulos afectados: Mercado — Subastas, Inventario.
*   Otros equipos: G08 (retenciones de varios días y devolución de muchas retenciones juntas); G02 (qué pasa al archivar un curso); PO (subasta de vidas y modelo de retención).
*   Impacto en datos / migraciones: tablas nuevas `auction`, `auction_bid` y `auction_pending_refund`, con los campos de auditoría y `is_active` de la guía de producto; `version` para el control de concurrencia.
*   Feature toggles / flags: sí. Uno para activar las subastas por ambiente y otro para usar el Banco real o el simulado. Se retiran cuando la integración con el Banco esté estable.

## Historias de Usuario Asociadas (12)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#578** | G11 — Lanzar un ítem a subasta | New | — | [US-578-lanzar-un-item-a-subasta.md](../users/US-578-lanzar-un-item-a-subasta.md) |
| **#579** | G11 — Ver las subastas abiertas de mi curso | New | — | [US-579-ver-las-subastas-abiertas-de-mi-curso.md](../users/US-579-ver-las-subastas-abiertas-de-mi-curso.md) |
| **#580** | G11 — Hacer una oferta en una subasta | New | — | [US-580-hacer-una-oferta-en-una-subasta.md](../users/US-580-hacer-una-oferta-en-una-subasta.md) |
| **#581** | G11 — Mejorar mi oferta | New | — | [US-581-mejorar-mi-oferta.md](../users/US-581-mejorar-mi-oferta.md) |
| **#582** | G11 — Enterarme al instante si me superaron | New | — | [US-582-enterarme-al-instante-si-me-superaron.md](../users/US-582-enterarme-al-instante-si-me-superaron.md) |
| **#583** | G11 — No ofertar dos veces por error | New | — | [US-583-no-ofertar-dos-veces-por-error.md](../users/US-583-no-ofertar-dos-veces-por-error.md) |
| **#584** | G11 — Entregar el ítem al ganador al cerrar la subasta | New | — | [US-584-entregar-el-item-al-ganador-al-cerrar-la-subasta.md](../users/US-584-entregar-el-item-al-ganador-al-cerrar-la-subasta.md) |
| **#585** | G11 — Devolver las monedas a quienes no ganaron | New | — | [US-585-devolver-las-monedas-a-quienes-no-ganaron.md](../users/US-585-devolver-las-monedas-a-quienes-no-ganaron.md) |
| **#586** | G11 — Cerrar una subasta sin ofertas | New | — | [US-586-cerrar-una-subasta-sin-ofertas.md](../users/US-586-cerrar-una-subasta-sin-ofertas.md) |
| **#587** | G11 — Cancelar una subasta | New | — | [US-587-cancelar-una-subasta.md](../users/US-587-cancelar-una-subasta.md) |
| **#588** | G11 — Cerrar subastas si el curso se archiva o el alumno se da de baja | New | — | [US-588-cerrar-subastas-si-el-curso-se-archiva-o-el-alumno-se-da-de-baja.md](../users/US-588-cerrar-subastas-si-el-curso-se-archiva-o-el-alumno-se-da-de-baja.md) |
| **#589** | G11 — Terminar cierres de subasta que quedaron a medias | New | — | [US-589-terminar-cierres-de-subasta-que-quedaron-a-medias.md](../users/US-589-terminar-cierres-de-subasta-que-quedaron-a-medias.md) |
