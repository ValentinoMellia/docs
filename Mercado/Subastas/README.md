# [G11 - Mercado] Módulo de Subastas (Épica E-07)
## Documentación Arquitectónica, Resiliencia y Contratos de Integración

Este directorio consolida el diseño técnico, la matriz de resiliencia y los contratos de eventos para el subsistema de **Subastas** en la plataforma distribuida de Aula Quest (Tema 09 - Mercado & Tema 08 - Banco).

---

### Índice de Documentos Técnicos

| Documento | Descripción |
| :--- | :--- |
| [**Documento Maestro de Arquitectura (HTML)**](./documento-arquitectura-subastas.html) | **Documento HTML interactivo integral** con navegación por secciones, las 3 opciones de arquitectura, matriz de resiliencia, contratos JSON desplegables, diagramas Mermaid y el visor Archify embebido. |
| [**01 · Análisis de Opciones de Arquitectura**](./01-analisis-opciones-arquitectura.md) | Análisis exhaustivo de 3 alternativas arquitectónicas para subastas con integración contable de `HOLD` en Banco. Trade-offs, complejidad matemática y recomendación del analista senior. |
| [**02 · Matriz de Fallos, Errores y Resiliencia**](./02-matriz-fallos-resiliencia-y-soluciones.md) | Los 5 errores letales a evitar (anti-sniping, desincronización de TTL, doble martillo concurrente, etc.) y soluciones de compensación para cada caída de microservicio. |
| [**03 · Contratos de Eventos e Idempotencia**](./03-contratos-eventos-e-idempotencia.md) | Canales sincrónicos (REST/SSE) y asincrónicos (Kafka). Estrategia de idempotencia de 3 capas (`processed_events`, Outbox Pattern, `@Version`) y contratos JSON completos con la envoltura oficial de 5 campos. |
| [**Flujo Interactivo Archify (HTML)**](./flujo-subasta-archify.html) | Diagrama de secuencia interactivo autocontenido, validado bajo estándar *showcase* de Archify. Permite navegar por vistas guiadas, filtrar por actores, alternar temas y exportar. |
| [**Especificación Fuente Archify (JSON)**](./flujo-subasta-archify.json) | Modelo de datos y definición vectorial del diagrama de secuencia generado por Archify. |

---

### Visualización del Flujo Interactivo
Para explorar el diagrama interactivo de secuencia generado con **Archify**:
1. Abrir directamente en el navegador el archivo [flujo-subasta-archify.html](./flujo-subasta-archify.html).
2. Utilizar el selector de **Vistas Guiadas** en la barra superior para recorrer:
   - *Vista 1: Oferta Inicial y Creación de HOLD*
   - *Vista 2: Mejora de Oferta e Incremento en Banco*
   - *Vista 3: Cierre Concurrente, Adjudicación y Liberación Masiva*
3. Inspeccionar las tarjetas interactivas de **Idempotencia**, **Contratos Kafka** y **Resiliencia**.
