# [G11 — Inventario, Equipamiento y Consumo del Alumno]

> **Taiga Ref:** #131 | **ID:** 367361
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Melina Yain Medina

## Descripción y Objetivos

Objetivo
--------

Que el ALUMNO tenga una mochila de ítems propia de cada curso, pueda decidir cuándo equiparlos y activarlos, y que sus protecciones se apliquen ante el resultado de un desafío, sin que Mercado tenga que conocer ni administrar el estado de otros dominios (vidas, XP).

Suposiciones y Restricciones
----------------------------

*   **Suposiciones:**
    *   Cada ítem del inventario es una **instancia individual, nunca un contador** (RF-REC-05): el uso único debe ser trazable hasta la entrega concreta que lo consumió.
    *   Las recompensas de un curso solo se usan en ese mismo curso (RF-REC-01).
    *   Mercado **gobierna la existencia y el estado** del ítem; **nunca aplica el efecto** — eso lo interpreta quien consulta (en el futuro, Motor de Desafíos y Roadmap).
    *   El ítem entra al inventario del ALUMNO por compra (épica de Catálogo/Compra) o, en esta etapa, se puede pre-cargar de forma simulada para poder construir y probar esta épica sin esperar esa integración.
    *   Mientras los servicios de Identidad, Cursos/Matrícula, Motor de Desafíos y Roadmap no estén disponibles (equipos en desarrollo inicial), se trabaja con un ALUMNO y un curso/cohorte simulados, y con un **vocabulario de verbos de efecto definido provisoriamente por el propio equipo de Mercado** (por ejemplo `ABSORBER_FALLO`, `MULTIPLICAR_XP`), documentado como supuesto a revisar cuando esos equipos existan y haya con quién acordarlo.
*   **Restricciones (legales/técnicas):**
    *   Nada se elimina físicamente: baja lógica en todas las entidades (RF-NFR-01).
    *   Un ítem consumido nunca vuelve a estar disponible.
    *   Solo puede haber **un ítem equipado por verbo de efecto**, no uno en total — para que nunca haya ambigüedad sobre cuál se consume ante un fallo.

Criterios de Aceptación a nivel Épico
-------------------------------------

*   El conjunto mínimo de historias permite el flujo extremo a extremo **el ALUMNO tiene un ítem en su inventario → lo equipa → se simula un desafío fallido → el ítem se consume y el sistema informa que el castigo debería neutralizarse**.
*   KPIs iniciales alcanzan: p95 de la resolución de efectos por debajo de 200 ms (medido internamente, sin consumidor real todavía); cero consumos duplicados en la prueba de concurrencia.
*   Sin regresiones críticas en el catálogo (que es quien entrega el ítem al inventario) ni en el resto de Mercado.
*   Observabilidad y alertas configuradas: log de cada consumo con su contexto (alumno, ítem, curso, resultado).
*   Documentación de uso y operación publicada: contrato del endpoint de resolución de efectos publicado (OpenAPI), aunque hoy no tenga un consumidor real del lado de Desafíos ni de Roadmap.

Dependencias / Impactos
-----------------------

*   **Servicios / APIs:** Servicio de Mercado (dueño). Servicio de Identidad, Servicio de Cursos y Matrícula, Motor de Desafíos y Roadmap — **todos simulados/mockeados**, ya que esos equipos están en desarrollo inicial.
*   **Módulos afectados:** Mercado (Inventario). Consume del módulo de Catálogo de la épica anterior para saber qué ítems existen.
*   **Otros equipos:** ninguno bloqueante hoy. A coordinar el vocabulario de verbos con Motor de Desafíos y Roadmap apenas tengan algo con qué integrar.
*   **Impacto en datos / migraciones:** crea la estructura para el inventario del alumno (ítem, alumno, curso/cohorte, estado, fecha de consumo) y una tabla de contextos ya procesados para garantizar idempotencia en el consumo.
*   **Feature toggles / flags:** no se requieren inicialmente.

## Historias de Usuario Asociadas (5)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#132** | G11 — Ver mi inventario del curso | New | — | [US-132-ver-mi-inventario-del-curso.md](../users/US-132-ver-mi-inventario-del-curso.md) |
| **#133** | G11 — Equipar y desequipar un ítem | New | — | [US-133-equipar-y-desequipar-un-item.md](../users/US-133-equipar-y-desequipar-un-item.md) |
| **#134** | G11 — Activar un consumible con vigencia temporal | New | — | [US-134-activar-un-consumible-con-vigencia-temporal.md](../users/US-134-activar-un-consumible-con-vigencia-temporal.md) |
| **#135** | G11 — Aplicar automáticamente mi protección ante un desafío fallido | New | — | [US-135-aplicar-automaticamente-mi-proteccion-ante-un-desafio-fallido.md](../users/US-135-aplicar-automaticamente-mi-proteccion-ante-un-desafio-fallido.md) |
| **#136** | G11 — Recibir un ítem sin comprarlo | New | — | [US-136-recibir-un-item-sin-comprarlo.md](../users/US-136-recibir-un-item-sin-comprarlo.md) |
