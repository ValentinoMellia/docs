# [G11 — Comprar una vida sin pasarme del tope]

> **Taiga Ref:** #142 | **ID:** 9539965
> **Épica:** [#137 — G11 — Compra Directa de Ítems del Mercado](../epics/EPIC-137-compra-directa-de-items-del-mercado.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** comprar una vida solo cuando tengo lugar para ella
*   **Para:** no gastar monedas en algo que el sistema después me va a rechazar

Notas / Observaciones
---------------------

*   **Reglas de negocio:** existe un máximo de vidas vigentes por curso, simulado con el valor de referencia PAR-12 del PRD (3 vidas) mientras Roadmap no exista.
*   **Validaciones:** la comprobación del tope ocurre **antes** de mover monedas.
*   **Datos obligatorios:** ID del ALUMNO, curso/cohorte, cantidad de vidas vigentes (simulada), tope vigente.
*   **Performance (tiempos, volumen, límites):** agrega una verificación al camino de compra; debe resolverse rápido al ser un valor simulado en esta etapa.
*   **Seguridad (roles, permisos, datos sensibles):** la verificación de vidas es sobre el propio ALUMNO.
*   **Accesibilidad (WCAG/teclado/lectores):** el motivo del rechazo por tope debe distinguirse claramente del rechazo por saldo.
*   **Otros:** cuando Roadmap exista, esta verificación pasa de un valor simulado a una consulta real — mismo patrón de puerto y adaptador que el resto del proyecto.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** si el ALUMNO está por debajo del máximo simulado de vidas, al comprar una vida se le acredita y su contador sube en uno.
*   **CA2:** si el ALUMNO ya alcanzó el máximo, la operación se rechaza antes de mover monedas, indicando que es por el tope.
*   **CA3:** el motivo de rechazo por tope se distingue explícitamente del motivo de rechazo por saldo insuficiente.
*   **Extras (opcional):** el catálogo muestra el ítem de vida atenuado cuando el ALUMNO ya está en el tope.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Compra de vidas con validación del tope vigente simulado por curso

**Escenario 1**

*   **Dado:** que el ALUMNO tiene menos vidas simuladas que el máximo permitido
*   **Cuando:** compra una vida
*   **Entonces:** se le descuenta el precio y su contador de vidas simuladas sube en uno

**Escenario 2**

*   **Dado:** que el ALUMNO tiene el máximo simulado de vidas vigentes
*   **Cuando:** intenta comprar una vida
*   **Entonces:** la operación se rechaza antes de mover monedas y el mensaje indica que alcanzó el tope

**Escenario 3**

*   **Dado:** los dos motivos de rechazo posibles (saldo y tope)
*   **Cuando:** se produce cada uno por separado
*   **Entonces:** el ALUMNO recibe mensajes claramente distintos para cada caso

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** reutiliza `POST /api/market/orders`, con la verificación de tope simulada internamente

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 5
*   **Prioridad (MoSCoW / Numérica):** Should / 2

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 5 | Debería / 2 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño, con el tope simulado internamente). Roadmap — futuro reemplazo del valor simulado.
*   **Módulos afectados:** Órdenes.
*   **Otros equipos / aprobaciones:** ninguna bloqueante hoy; a coordinar con Roadmap cuando exista, para decidir si la validación pasa a ser una reserva de cupo o una simple consulta.
*   **Impacto en datos / migraciones:** ninguno adicional en esta etapa, salvo el valor simulado del tope.
*   **Riesgos y mitigación (opcional):** cuando Roadmap exista, puede aparecer una ventana de carrera entre validar el tope y acreditar la vida. Mitigación: queda documentado para resolver en esa integración, no bloquea esta historia hoy.

