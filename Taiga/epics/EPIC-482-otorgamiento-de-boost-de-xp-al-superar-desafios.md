# [G11 — Otorgamiento de Boost de XP al superar desafíos]

> **Taiga Ref:** #482 | **ID:** 367760
> **Estado:** New | **Asignado a:** Sin asignar
> **Propietario:** Juan Bosque

## Descripción y Objetivos

# [G11] — Otorgamiento de Boost de XP al superar desafíos

---

## Objetivo

Permitir que el alumno obtenga puntos de experiencia adicionales al superar exitosamente un desafío, aplicando el multiplicador de XP correspondiente para acelerar su progreso dentro de la plataforma.

---

## Suposiciones y Restricciones

- Suposiciones:
  - El desafío tiene definidos sus puntos base de experiencia.
  - El desafío puede tener un multiplicador de XP configurado.
  - El multiplicador se aplica únicamente cuando el alumno supera el desafío.
  - La acreditación de XP se realiza una única vez por desafío superado.
  - Los reintentos de un desafío previamente superado pueden realizarse con fines de práctica, pero no generan XP adicional ni vuelven a aplicar el multiplicador.
  - La resolución del desafío permite determinar si fue aprobado o reprobado.
  - La identidad del alumno y la pertenencia del intento al alumno autenticado pueden ser verificadas.

- Restricciones (legales/técnicas):
  - No se debe acreditar XP ni aplicar el multiplicador cuando el desafío no es superado.
  - No se debe duplicar la acreditación de XP ante reintentos de un desafío previamente superado.
  - El cálculo de XP debe realizarse al momento de finalizar la resolución del desafío.
  - La comunicación entre los servicios involucrados debe respetar los contratos definidos para la arquitectura de integración.

---

## Criterios de Aceptación a nivel Épico

- El conjunto mínimo de historias permite completar el flujo extremo a extremo de resolución de un desafío, determinación del resultado y acreditación de la XP correspondiente.
- Cuando el alumno supera un desafío por primera vez, se acredita la XP resultante de aplicar el multiplicador correspondiente sobre los puntos base.
- Cuando el alumno no supera el desafío, se acreditan 0 puntos de experiencia.
- Cuando el alumno vuelve a realizar un desafío que ya había superado, se permite la práctica pero no se acreditan puntos adicionales ni se vuelve a aplicar el multiplicador.
- La XP acreditada queda registrada para mantener trazabilidad de la experiencia obtenida por el alumno y evitar duplicaciones.
- No se producen regresiones críticas en la resolución de desafíos ni en la actualización del progreso o experiencia del alumno.
- Se dispone de logs que permitan identificar el resultado de la resolución y la acreditación de XP asociada al desafío.

---

## Dependencias / Impactos

- Servicios / APIs:
  - `challenge-service`
  - `gamification-service`

- Módulos afectados:
  - Motor de cálculo de XP.
  - Resolución y finalización de desafíos.
  - Perfil y progreso del alumno.
  - Historial de intentos y desafíos superados.

- Otros equipos:
  - No especificado.

- Impacto en datos / migraciones:
  - Registro de la XP otorgada por desafío.
  - Registro del estado de superación del desafío para evitar acreditaciones duplicadas.
  - Historial de intentos asociado al alumno y al desafío.

- Feature toggles / flags:
  - No especificado.

## Historias de Usuario Asociadas (2)

| Ref | Título | Estado | Puntos | Archivo |
| :---: | :--- | :---: | :---: | :--- |
| **#130** | G11-HU06 — Obtener multiplicador de experiencia (XP Boost) al superar un desafío | New | 1 | [US-130-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md](../users/US-130-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md) |
| **#810** | G11-HU06 — Obtener multiplicador de experiencia (XP Boost) al superar un desafío | New | — | [US-810-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md](../users/US-810-obtener-multiplicador-de-experiencia-xp-boost-al-superar-un-desafio.md) |
