# [AulaQuest] Extracción y Backlog de Taiga · Grupo 11 (Mercado)

Este directorio contiene la exportación directa y sincronizada de todas las **Épicas** e **Historias de Usuario** pertenecientes al Grupo 11 (G11) extraídas desde el proyecto en Taiga.

> **Contexto de negocio:** [`../CONTEXTO-MERCADO-SPRINT1.md`](../CONTEXTO-MERCADO-SPRINT1.md) es el documento que reconcilia todo este backlog con las decisiones cerradas del equipo, y está preparado explícitamente para servir de contexto a un futuro ciclo SDD (`sdd-explore → sdd-propose → sdd-spec → sdd-design → sdd-tasks → sdd-apply`) — todavía no se ejecuta ese ciclo, pero cuando se haga, ese es el punto de partida.

## Resumen de Épicas (6)

| Ref | ID | Título de la Épica | HUs Asociadas | Estado | Documento |
| :---: | :---: | :--- | :---: | :---: | :--- |
| **#90** | 367203 | G11 — Catálogo Abierto por Plantillas | 7 | New | [EPIC-090-gestion-del-catalogo-de-mercado.md](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) |
| **#131** | 367361 | G11 — Inventario, Equipamiento y Consumo del Alumno | 5 | New | [EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) |
| **#137** | 367363 | G11 — Compra Directa con Doble Reserva | 6 | New | [EPIC-137-compra-directa-de-items-del-mercado.md](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) |
| **#482** | 367760 | G11 — Otorgamiento de Boost de XP al superar desafíos | 2 | New | [EPIC-482-otorgamiento-de-boost-de-xp-al-superar-desafios.md](./epics/EPIC-482-otorgamiento-de-boost-de-xp-al-superar-desafios.md) |
| **#577** | 368163 | G11 — Subastas de Ítems con Tiempo Límite | 12 | New | [EPIC-577-subastas-de-items-con-tiempo-limite.md](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) |
| **#770** | 368559 | G11 — Vencimiento de Ítems del Inventario | 5 | New | [EPIC-770-vencimiento-de-items-del-inventario.md](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) |

## Resumen de Historias de Usuario (37)

> Nota: #945 y #946 se sumaron en Taiga el 17/09/2026 (descomposición fina de la épica #90) y todavía no tienen archivo local exportado.

| Ref | ID | Título | Épica Asociada | Tareas | Estado | Documento |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| **#92** | 9535755 | G11 — Publicar una oferta en mi curso a partir de una plantilla | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | [US-092-crear-un-item.md](./users/US-092-crear-un-item.md) |
| **#94** | 9535762 | G11 — Consultar la vitrina de mi curso | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | [US-094-consultar-catalogo-disponible.md](./users/US-094-consultar-catalogo-disponible.md) |
| **#95** | 9535768 | G11 — Editar una oferta publicada | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | [US-095-modificar-un-item.md](./users/US-095-modificar-un-item.md) |
| **#96** | 9535772 | G11 — Activar o desactivar una oferta | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | [US-096-activar-o-desactivar-un-item.md](./users/US-096-activar-o-desactivar-un-item.md) |
| **#98** | 9535782 | G11 — Consultar el detalle de una oferta | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | [US-098-consultar-detalle-de-un-articulo.md](./users/US-098-consultar-detalle-de-un-articulo.md) |
| **#945** | 9555132 | G11 — Listar las plantillas base disponibles | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | *(sin archivo local — creada en Taiga el 17/09/2026)* |
| **#946** | 9555133 | G11 — Ver el catálogo completo de mi curso | [#90](./epics/EPIC-090-gestion-del-catalogo-de-mercado.md) | 0 | New | *(sin archivo local — creada en Taiga el 17/09/2026)* |
| **#130** | 367359 | G11-HU06 — Obtener multiplicador de experiencia (XP Boost) al superar un desafío | [#482](./epics/EPIC-482-otorgamiento-de-boost-de-xp-al-superar-desafios.md) | 0 | New | [US-130-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md](./users/US-130-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md) |
| **#132** | 9539908 | G11 — Ver mi inventario del curso | [#131](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) | 7 | New | [US-132-ver-mi-inventario-del-curso.md](./users/US-132-ver-mi-inventario-del-curso.md) |
| **#133** | 9539931 | G11 — Equipar y desequipar un ítem | [#131](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) | 0 | New | [US-133-equipar-y-desequipar-un-item.md](./users/US-133-equipar-y-desequipar-un-item.md) |
| **#134** | 9539944 | G11 — Activar un consumible con vigencia temporal | [#131](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) | 0 | New | [US-134-activar-un-consumible-con-vigencia-temporal.md](./users/US-134-activar-un-consumible-con-vigencia-temporal.md) |
| **#135** | 9539946 | G11 — Aplicar automáticamente mi protección ante un desafío fallido | [#131](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) | 0 | New | [US-135-aplicar-automaticamente-mi-proteccion-ante-un-desafio-fallido.md](./users/US-135-aplicar-automaticamente-mi-proteccion-ante-un-desafio-fallido.md) |
| **#136** | 9539948 | G11 — Recibir un ítem sin comprarlo | [#131](./epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md) | 0 | New | [US-136-recibir-un-item-sin-comprarlo.md](./users/US-136-recibir-un-item-sin-comprarlo.md) |
| **#138** | 9539955 | G11 — Comprar una oferta del catálogo | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 6 | New | [US-138-comprar-un-item-del-catalogo.md](./users/US-138-comprar-un-item-del-catalogo.md) |
| **#139** | 9539961 | G11 — No pagar dos veces por un doble clic | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 4 | New | [US-139-no-pagar-dos-veces-por-un-doble-clic.md](./users/US-139-no-pagar-dos-veces-por-un-doble-clic.md) |
| **#140** | 9539963 | G11 — Ver el resultado de una compra en proceso | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 0 | New | [US-140-ver-el-resultado-de-una-compra-en-proceso.md](./users/US-140-ver-el-resultado-de-una-compra-en-proceso.md) |
| **#141** | 9539964 | G11 — Recuperar mis monedas si la compra no se completó | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 0 | New | [US-141-recuperar-mis-monedas-si-la-compra-no-se-completo.md](./users/US-141-recuperar-mis-monedas-si-la-compra-no-se-completo.md) |
| **#142** | 9539965 | G11 — Comprar una vida sin pasarme del tope | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 3 | New | [US-142-comprar-una-vida-sin-pasarme-del-tope.md](./users/US-142-comprar-una-vida-sin-pasarme-del-tope.md) |
| **#143** | 9539966 | G11 — Revisar las compras que quedaron a medias | [#137](./epics/EPIC-137-compra-directa-de-items-del-mercado.md) | 0 | New | [US-143-revisar-las-compras-que-quedaron-a-medias.md](./users/US-143-revisar-las-compras-que-quedaron-a-medias.md) |
| **#578** | 9549018 | G11 — Lanzar un ítem a subasta | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-578-lanzar-un-item-a-subasta.md](./users/US-578-lanzar-un-item-a-subasta.md) |
| **#579** | 9549019 | G11 — Ver las subastas abiertas de mi curso | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-579-ver-las-subastas-abiertas-de-mi-curso.md](./users/US-579-ver-las-subastas-abiertas-de-mi-curso.md) |
| **#580** | 9549020 | G11 — Hacer una oferta en una subasta | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-580-hacer-una-oferta-en-una-subasta.md](./users/US-580-hacer-una-oferta-en-una-subasta.md) |
| **#581** | 9549021 | G11 — Mejorar mi oferta | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-581-mejorar-mi-oferta.md](./users/US-581-mejorar-mi-oferta.md) |
| **#582** | 9549022 | G11 — Enterarme al instante si me superaron | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-582-enterarme-al-instante-si-me-superaron.md](./users/US-582-enterarme-al-instante-si-me-superaron.md) |
| **#583** | 9549023 | G11 — No ofertar dos veces por error | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-583-no-ofertar-dos-veces-por-error.md](./users/US-583-no-ofertar-dos-veces-por-error.md) |
| **#584** | 9549024 | G11 — Entregar el ítem al ganador al cerrar la subasta | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-584-entregar-el-item-al-ganador-al-cerrar-la-subasta.md](./users/US-584-entregar-el-item-al-ganador-al-cerrar-la-subasta.md) |
| **#585** | 9549025 | G11 — Devolver las monedas a quienes no ganaron | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-585-devolver-las-monedas-a-quienes-no-ganaron.md](./users/US-585-devolver-las-monedas-a-quienes-no-ganaron.md) |
| **#586** | 9549026 | G11 — Cerrar una subasta sin ofertas | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-586-cerrar-una-subasta-sin-ofertas.md](./users/US-586-cerrar-una-subasta-sin-ofertas.md) |
| **#587** | 9549027 | G11 — Cancelar una subasta | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-587-cancelar-una-subasta.md](./users/US-587-cancelar-una-subasta.md) |
| **#588** | 9549028 | G11 — Cerrar subastas si el curso se archiva o el alumno se da de baja | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-588-cerrar-subastas-si-el-curso-se-archiva-o-el-alumno-se-da-de-baja.md](./users/US-588-cerrar-subastas-si-el-curso-se-archiva-o-el-alumno-se-da-de-baja.md) |
| **#589** | 9549029 | G11 — Terminar cierres de subasta que quedaron a medias | [#577](./epics/EPIC-577-subastas-de-items-con-tiempo-limite.md) | 0 | New | [US-589-terminar-cierres-de-subasta-que-quedaron-a-medias.md](./users/US-589-terminar-cierres-de-subasta-que-quedaron-a-medias.md) |
| **#778** | 9552674 | G11 — Configurar el vencimiento de un ítem del curso | [#770](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) | 0 | New | [US-778-configurar-el-vencimiento-de-un-item-del-curso.md](./users/US-778-configurar-el-vencimiento-de-un-item-del-curso.md) |
| **#785** | 9552675 | G11 — Saber cuánto dura un ítem antes de comprarlo | [#770](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) | 0 | New | [US-785-ver-cuando-vencen-mis-items.md](./users/US-785-ver-cuando-vencen-mis-items.md) |
| **#786** | 9552676 | G11 — Recibir un aviso antes de que venza un ítem | [#770](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) | 0 | New | [US-786-recibir-un-aviso-antes-de-que-venza-un-item.md](./users/US-786-recibir-un-aviso-antes-de-que-venza-un-item.md) |
| **#787** | 9552679 | G11 — Vencer automáticamente los ítems no usados | [#770](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) | 0 | New | [US-787-vencer-automaticamente-los-items-no-usados.md](./users/US-787-vencer-automaticamente-los-items-no-usados.md) |
| **#788** | 9552680 | G11 — Consultar los ítems vencidos del curso | [#770](./epics/EPIC-770-vencimiento-de-items-del-inventario.md) | 0 | New | [US-788-consultar-los-items-vencidos-del-curso.md](./users/US-788-consultar-los-items-vencidos-del-curso.md) |
| **#810** | 9552739 | G11-HU06 — Obtener multiplicador de experiencia (XP Boost) al superar un desafío | [#482](./epics/EPIC-482-otorgamiento-de-boost-de-xp-al-superar-desafios.md) | 0 | New | [US-810-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md](./users/US-810-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md) |
