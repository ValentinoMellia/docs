# [G11 — Recibir un ítem sin comprarlo]

> **Taiga Ref:** #136 | **ID:** 9539948
> **Épica:** [#131 — G11 — Inventario, Equipamiento y Consumo del Alumno](../epics/EPIC-131-inventario-equipamiento-y-consumo-del-alumno.md)
> **Estado:** New | **Puntos:** —
> **Asignado a:** Sin asignar | **Propietario:** Melina Yain Medina

## Detalle / Especificación (Taiga)

Descripción (Como / Quiero / Para)
----------------------------------

*   **Como:** ALUMNO
*   **Quiero:** recibir un ítem otorgado por un logro especial
*   **Para:** poder usarlo igual que uno comprado

Notas / Observaciones
---------------------

*   **Reglas de negocio:** hoy no existe ningún servicio (Motor de Desafíos, Roadmap) capaz de disparar este otorgamiento, porque ambos están en desarrollo inicial. El ítem otorgado debe guardar su origen, para distinguir en el futuro lo comprado de lo regalado.
*   **Validaciones:** solo un conjunto acotado de orígenes debería poder otorgar; hoy se simula con un origen fijo de prueba.
*   **Datos obligatorios:** ID del ALUMNO, curso/cohorte, ítem, origen, motivo.
*   **Performance (tiempos, volumen, límites):** volumen bajo y esporádico.
*   **Seguridad (roles, permisos, datos sensibles):** toda entrega debe quedar auditada con su motivo.
*   **Accesibilidad (WCAG/teclado/lectores):** el aviso de ítem recibido debe ser perceptible sin depender solo de una animación.
*   **Otros:** esta historia queda documentada pero **no se recomienda tomarla en este sprint**: sin Motor de Desafíos ni Roadmap, no hay ningún origen real que otorgue nada, y construirla ahora sería simular por ambos lados algo que hoy no tiene ningún consumidor.

Criterios de Aceptación (CA)
----------------------------

*   **CA1:** un origen autorizado puede otorgar un ítem al inventario del ALUMNO en estado disponible.
*   **CA2:** un mismo motivo de otorgamiento procesado dos veces entrega el ítem una sola vez.
*   **CA3:** el ítem otorgado registra cómo se obtuvo, distinto de uno comprado.
*   **Extras (opcional):** si el ALUMNO ya alcanzó el máximo permitido de ese ítem, el otorgamiento se rechaza y queda registrado.

BDD (mínimo 3 escenarios)
-------------------------

**Característica:** Alta de ítems en el inventario por canales distintos de la compra

**Escenario 1**

*   **Dado:** un origen autorizado que otorga un ítem al ALUMNO
*   **Cuando:** se procesa el otorgamiento
*   **Entonces:** el ítem aparece disponible en el inventario del ALUMNO

**Escenario 2**

*   **Dado:** que ya se otorgó un ítem por un motivo determinado
*   **Cuando:** el otorgamiento se procesa nuevamente con el mismo motivo
*   **Entonces:** el ALUMNO no recibe un segundo ítem

**Escenario 3**

*   **Dado:** un ALUMNO con un ítem comprado y otro otorgado
*   **Cuando:** consulta el detalle de su inventario
*   **Entonces:** cada instancia indica cómo fue obtenida

Prototipo
---------

*   **Capturas:** \[PEGAR AQUÍ\]
*   **URL Figma:** \[pendiente\]
*   **Libro de cuentos:** \[pendiente\]
*   **API simulada / Swagger:** `POST /api/market/inventory/grant`

Estimación / Prioridad
----------------------

**Formato rápido**

*   **Puntos (Fibonacci):** 8
*   **Prioridad (MoSCoW / Numérica):** Won't (por ahora) / 5

**Formato tabla (opcional)**

| Puntos (Fibonacci) | Prioridad (MoSCoW / Numérica) |
| --- | --- |
| 8 | No lo hará por ahora / 5 |

Dependencias / Impactos
-----------------------

*   **Servicios involucrados:** Mercado (dueño). Motor de Desafíos y Roadmap como futuros otorgantes.
*   **Módulos afectados:** Mercado (Inventario).
*   **Otros equipos / aprobaciones:** definir con Motor de Desafíos y Roadmap, cuando existan, quién puede otorgar y con qué garantía de no duplicación.
*   **Impacto en datos / migraciones:** agrega campo de origen y de motivo al ítem de inventario.
*   **Riesgos y mitigación (opcional):** construir esta historia hoy generaría trabajo descartable si el mecanismo real termina siendo distinto al simulado. Mitigación: dejarla documentada y no tomarla hasta que exista al menos un origen real.

