# [G11-HU06 — Obtener multiplicador de experiencia (XP Boost) al superar un desafío]

> **Taiga Ref:** #130 | **ID:** 367359
> **Épica:** [#482 — G11 — Otorgamiento de Boost de XP al superar desafíos](../epics/EPIC-482-otorgamiento-de-boost-de-xp-al-superar-desafios.md)
> **Estado:** New | **Puntos:** 1
> **Asignado a:** Sin asignar | **Propietario:** Juan Bosque

## Detalle / Especificación (Taiga)

# [G11-HU06] — Obtener multiplicador de experiencia (XP Boost) al superar un desafío

---

## Descripción (Como / Quiero / Para)
**Como:** Alumno  
**Quiero:** recibir un multiplicador de puntos de experiencia (XP Boost) al superar un desafío específico  
**Para:** aumentar mi nivel de experiencia de forma acelerada dentro de la plataforma  

---

## Notas / Observaciones
**Reglas de negocio:**
- Cada desafío posee su propio valor de multiplicador de XP configurado (ej. 1.2x, 1.5x, 2x) o carece de él según su diseño.
- Si el alumno no supera o falla el desafío, se otorgan 0 puntos de experiencia (no se acredita XP base ni multiplicador).
- Se aplica el multiplicador asignado al desafío sobre los puntos base del mismo únicamente cuando la condición de aprobación sea alcanzada.

**Validaciones:**
- El cálculo se valida al momento de enviar la resolución del desafío.

**Datos obligatorios:** ID del alumno, ID del desafío, estado de resolución (aprobado/reprobado), puntos base del desafío, valor del multiplicador.  
**Performance:** no aplica (cálculo en tiempo real durante la finalización del intento).  
**Seguridad:** verificación de que el intento sea válido y pertenezca al alumno autenticado.  
**Accesibilidad:** no aplica.  
**Otros:** no otorga XP repetida en reintentos de desafíos previamente superados.

---

## Criterios de Aceptación (CA)
- **CA1:** Se calcula el XP total multiplicando los puntos base por el boost configurado solo si el desafío es superado por primera vez.
- **CA2:** Si el intento no alcanza el puntaje mínimo de aprobación, se acreditan 0 puntos de XP.
- **CA3:** Si se reintenta un desafío previamente superado, no se acreditan puntos adicionales ni se vuelve a aplicar el multiplicador.

---

## BDD (mínimo 3 escenarios)
**Característica:** Otorgamiento de multiplicador de experiencia por desafío

**Escenario 1:** Desafío superado con éxito (Camino Feliz)  
**Dado:** que estoy resolviendo un desafío que tiene un multiplicador de XP asignado  
**Cuando:** lo completo superando el puntaje mínimo requerido  
**Entonces:** el sistema debe calcular los puntos base multiplicados por el valor del boost configurado para ese desafío  
**Y:** mostrar un mensaje notificando la cantidad total de puntos obtenidos y el nivel de experiencia actualizado  

**Escenario 2:** Desafío no superado / Fallado (Sin acreditación)  
**Dado:** que estoy resolviendo un desafío que tiene un multiplicador de XP asignado  
**Cuando:** finalizo el intento sin alcanzar el puntaje mínimo de aprobación  
**Entonces:** el sistema debe indicar que el desafío no fue superado  
**Y:** acreditar 0 puntos de experiencia en el perfil  

**Escenario 3:** Reintento de un desafío previamente superado  
**Dado:** que ya he completado y superado exitosamente un desafío en el pasado  
**Cuando:** decido resolverlo nuevamente para practicar  
**Entonces:** el sistema debe permitir realizar la práctica  
**Y:** notificar que el multiplicador de XP de ese desafío ya fue otorgado previamente y no sumará puntos adicionales  

---

## Prototipo
**Mock API / Swagger:**
- Cálculo e integración con el servicio de gamificación / experiencia del alumno (`gamification-service`).

---

## Estimación / Prioridad
- **Puntos (Fibonacci):** 1
- **Prioridad (MoSCoW / Numérica):** Should

---

## Dependencias / Impactos
- **Servicios involucrados:** `gamification-service`, `challenge-service`
- **Módulos afectados:** motor de cálculo de XP, perfil del alumno, historial de intentos
- **Otros equipos / aprobaciones:** no aplica
- **Impacto en datos / migraciones:** tablas de registro de XP otorgada por desafío e historial de superación
- **Riesgos y mitigación (opcional):** riesgo de duplicación de XP en reintentos; mitigado mediante control de estado previo en la BD.

