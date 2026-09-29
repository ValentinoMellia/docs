# [G11 — Compra Directa de Ítems del Mercado]

> **Taiga Ref:** #137 | **ID:** 367363
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Melina Yain Medina

## Descripción y Objetivos

Objetivo
--------

Que el ALUMNO pueda canjear monedas por un ítem publicado en el catálogo de su curso, con la garantía de que si algo falla a mitad de camino no termina pagando por algo que no recibió — cerrando así el ciclo completo Catálogo → Compra → Inventario.

Suposiciones y Restricciones
----------------------------

*   **Suposiciones:**
    *   El catálogo (épica anterior) ya existe y expone ítems con su precio.
    *   El inventario (épica anterior) ya existe y puede recibir la acreditación de un ítem.
    *   **Mientras el Banco (servicio de monedas) no esté disponible, Mercado implementa un saldo simulado propio y transitorio** (un registro interno de monedas por alumno y curso/cohorte), para poder construir y demostrar el flujo de compra de punta a punta. Este mecanismo se diseña detrás de una interfaz propia (puerto), de modo que reemplazarlo por la integración real del Banco sea escribir una clase de infraestructura y no tocar la lógica de dominio.
    *   El tope de vidas vigentes (necesario para la compra de una vida extra) se simula con un valor fijo (alineado al parámetro de referencia PAR-12 del PRD: 3 vidas), mientras Roadmap no exista.
*   **Restricciones (legales/técnicas):**
    *   Las monedas solo se canjean por vidas o equipamiento, nunca al revés (RF-INT-01).
    *   Las monedas de un curso solo sirven en ese curso (RF-INT-04); no existe saldo global.
    *   Nada se elimina físicamente: baja lógica en todas las entidades (RF-NFR-01).
    *   Está prohibido el patrón "descontar y después entregar": si algo se cobra y no se entrega, hay que compensar (devolver) siempre.
    *   La orden persiste el precio con el que se ejecutó, porque un cambio de parámetro rige solo hacia adelante (RF-CFG-06) — un ajuste de precio no debe alterar compras ya realizadas.

Criterios de Aceptación a nivel Épico
-------------------------------------

*   El conjunto mínimo de historias permite el flujo extremo a extremo **el ALUMNO confirma la compra → se le descuentan las monedas (del saldo simulado) → recibe el ítem en su inventario**, y su contrapartida **la entrega falla → recupera las monedas**.
*   KPIs iniciales alcanzan: cero cobros duplicados en la prueba de reintento; cero órdenes cobradas sin ítem ni devolución al cerrar la prueba de fallas inyectadas; 100% de las órdenes con precio persistido.
*   Sin regresiones críticas en el catálogo ni en el inventario, que es donde termina la compra.
*   Observabilidad y alertas configuradas: traza por orden de punta a punta; alerta cuando una orden queda pendiente de compensación.
*   Documentación de uso y operación publicada: OpenAPI de los endpoints de compra y diagrama de estados de la orden.

Dependencias / Impactos
-----------------------

*   **Servicios / APIs:** Servicio de Mercado (dueño, incluido el saldo simulado). Servicio de Identidad, Servicio de Cursos y Matrícula, Banco y Roadmap — **todos mockeados/simulados**, ya que esos equipos están en desarrollo inicial.
*   **Módulos afectados:** Mercado (Catálogo, Órdenes, Inventario).
*   **Otros equipos:** ninguno bloqueante hoy. Cuando Banco exista, se reemplaza el saldo simulado por la integración real detrás del mismo puerto.
*   **Impacto en datos / migraciones:** crea la estructura de órdenes (con precio aplicado, estado y clave de idempotencia) y la del saldo simulado por alumno y curso/cohorte.
*   **Feature toggles / flags:** sí, uno para alternar entre el saldo simulado y la futura integración real con Banco, sin necesidad de desplegar. Plan de retiro: se elimina cuando la integración real esté estable.

## Historias de Usuario Asociadas (6)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#138** | G11 — Comprar un ítem del catálogo | New | — | [US-138-comprar-un-item-del-catalogo.md](../users/US-138-comprar-un-item-del-catalogo.md) |
| **#139** | G11 — No pagar dos veces por un doble clic | New | — | [US-139-no-pagar-dos-veces-por-un-doble-clic.md](../users/US-139-no-pagar-dos-veces-por-un-doble-clic.md) |
| **#140** | G11 — Ver el resultado de una compra en proceso | New | — | [US-140-ver-el-resultado-de-una-compra-en-proceso.md](../users/US-140-ver-el-resultado-de-una-compra-en-proceso.md) |
| **#141** | G11 — Recuperar mis monedas si la compra no se completó | New | — | [US-141-recuperar-mis-monedas-si-la-compra-no-se-completo.md](../users/US-141-recuperar-mis-monedas-si-la-compra-no-se-completo.md) |
| **#142** | G11 — Comprar una vida sin pasarme del tope | New | — | [US-142-comprar-una-vida-sin-pasarme-del-tope.md](../users/US-142-comprar-una-vida-sin-pasarme-del-tope.md) |
| **#143** | G11 — Revisar las compras que quedaron a medias | New | — | [US-143-revisar-las-compras-que-quedaron-a-medias.md](../users/US-143-revisar-las-compras-que-quedaron-a-medias.md) |
