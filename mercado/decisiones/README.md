# Decisiones del grupo sobre Mercado

Aquí se guardan los archivos que exporta el taller de decisiones ([`../estado-actual/taller-decisiones.html`](../estado-actual/taller-decisiones.html)) y el registro de lo que se definió.

## Cómo se usa

1. Abrir `taller-decisiones.html` en el navegador (doble clic; no requiere servidor ni internet).
2. En la reunión, recorrer las tarjetas: **decisiones** (`D*`, `S*`, `T*`) con opciones y una marcada como recomendada, y **tareas** (`K*`) para aceptar, diferir o descartar, con responsable y sprint.
3. **Exportar JSON** (fuente de verdad) y, si sirve, **Exportar Markdown** (lectura humana).
4. Guardar el archivo en esta carpeta con el nombre `decisiones-mercado-AAAA-MM-DD.json` y abrir un PR.
5. Pasar el JSON a Claude en el chat: lo usa como contexto para actualizar la documentación y proponer los pasos técnicos por repo.

Las respuestas se guardan solo en el navegador de quien las carga; para juntar las de varias personas, cada una exporta su JSON y se **importan** en una sola copia (se conserva la respuesta más reciente de cada tarjeta).

## Formato del JSON (`mercado-decisiones/v1`)

| Campo | Contenido |
|---|---|
| `snapshot` | Commits analizados (`tpi-market`, `tpi-accounting`) |
| `meeting` | `date`, `participants` |
| `summary` | Totales: tratadas, pendientes, decisiones tomadas |
| `items[]` | `id`, `kind` (`decision` \| `task`), `area`, `priority`, `title`, `status`, `chosenOptionId`, `chosenOptionLabel`, `recommendedOptionId`, `customOption`, `notes`, `owner`, `suggestedOwner`, `targetSprint`, `updatedAt` |

Estados: decisiones → `pending`, `done`, `deferred`, `info` (falta información); tareas → `pending`, `done` (aceptada), `deferred`, `rejected`.

## Registro

_Todavía no hay decisiones exportadas._
