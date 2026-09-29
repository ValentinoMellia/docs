# Mercado (Tema 09): estado actual y documentación

Mercado es el microservicio (`tpi-market`) que orquesta la **tienda** de una plataforma gamificada: los profesores publican ítems de equipamiento (escudos, boosts, vidas) para su curso y los alumnos los compran o, desde el Sprint 02, los subastan con monedas. Mercado **no guarda** monedas ni inventario: eso es de **Accounting** (`tpi-accounting`, ex "Banco/Inventario").

> **Estado al 29/09/2026** · `tpi-market` `develop` @ `7528610` · `tpi-accounting` `develop` @ `f965420`. Todo fue verificado leyendo el código de ambos repos.

## Estado por función

| Función | Qué es | Estado en el código | Detalle |
|---|---|---|---|
| **Gestión de tienda** | El profesor publica, edita, activa y desactiva ofertas basadas en plantillas | ✅ Implementada (con un bug de stock: `units_sold` no se incrementa) | [gestion-de-tienda.md](./estado-actual/gestion-de-tienda.md) |
| **Compra** | El alumno compra una oferta: hold de monedas → ítem al inventario → cobro | ⚠️ Saga completa con contrapartes simuladas; **no interopera con Accounting real** | [compra.md](./estado-actual/compra.md) |
| **Subasta** (Sprint 02) | El profesor subasta un ítem; los alumnos ofertan con monedas retenidas | ❌ Solo diseño; Accounting cubre buena parte del lado de monedas | [subasta.md](./estado-actual/subasta.md) |

## Lo más importante hoy

1. **Mercado y Accounting no pueden completar una compra entre sí**: topics distintos, `orderId` no UUID, comandos de hold apagados en Accounting, paso de aprovisionamiento sin equivalente y catálogo de ítems placeholder. Detalle y plan en [`estado-integracion-mercado-accounting.md`](../integracion/banco/estado-integracion-mercado-accounting.md).
2. Matrícula y docente del curso son **mocks** en todos los perfiles.
3. Para arrancar la subasta hay que cerrar primero el P0 de la compra y ajustar el diseño a lo que Accounting realmente acepta.

Lista priorizada completa: [`brechas-y-pendientes.md`](./estado-actual/brechas-y-pendientes.md).

## Mapa de documentos

| Necesito… | Leer |
|---|---|
| Entender el estado real de cada función | `estado-actual/` (esta carpeta) |
| Ver el contrato y los desajustes con Accounting | [`integracion/banco/estado-integracion-mercado-accounting.md`](../integracion/banco/estado-integracion-mercado-accounting.md) |
| Entender cómo viaja una petición hasta Mercado | [`arquitectura/flujo-de-una-peticion.md`](../arquitectura/flujo-de-una-peticion.md) |
| Diseño funcional del catálogo por plantillas | [`catalogos/README.md`](./catalogos/README.md) (parcialmente desactualizado) |
| Diseño técnico de subastas | [`subastas/README.md`](./subastas/README.md) (diseño de Fase 3) |
| Decisiones del equipo en Sprint 1 | [`arquitectura/CONTEXTO-MERCADO-SPRINT1.md`](../arquitectura/CONTEXTO-MERCADO-SPRINT1.md) |
| Historias y épicas | [`gestion/taiga/`](../gestion/taiga/README.md) |

## Regla para leer estos documentos

Cuando un documento de diseño (`catalogos/`, `subastas/`, `integracion/banco/flujo-*`) contradiga a `estado-actual/`, **manda `estado-actual/`**: describe lo que hace el código hoy. Los de diseño describen lo que se quería construir.
