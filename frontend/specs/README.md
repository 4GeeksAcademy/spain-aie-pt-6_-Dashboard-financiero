# Especificaciones del dashboard

Documentación únicamente: no implementa React, transporte HTTP ni tipos nuevos.
Contratos revisados en [routes.py](../../backend/app/routes.py) y registro de la
API en [main.py](../../backend/app/main.py). No se han ejecutado pruebas.

- [Componentes y estados](components.md)
- [Filtros de fechas](feature-1-date-filters.md)
- [Tabla de alertas](feature-2-alerts-table.md)
- [Comparativa B2B/B2C](feature-3-b2b-b2c-comparison.md)

## Convenciones

Todos los endpoints son `GET`; parámetros en query. Fechas `YYYY-MM-DD`, con
inicio y fin inclusivos y opcionales salvo indicación contraria.
`operation_type`: `income` o `outcome`; `business_type`: `B2B` o `B2C`.
`category`: `suppliers`, `sales`, `operational`, `administrative`, `others`.
`group_by`: `day`, `week`, `month` (por defecto `month`).
Los importes son números sin moneda declarada. Las listas no llevan envoltorio
ni paginación; una consulta sin resultados devuelve `[]`.

## Endpoints, parámetros y respuestas

| Endpoint | Parámetros | Respuesta de éxito |
| --- | --- | --- |
| `/api/metrics/facets` | Ninguno. | `{operation_types, business_types, categories, min_date, max_date}`; opciones como listas y límites como fechas. |
| `/api/metrics` | Fechas, `category`, `operation_type`, todos opcionales. No admite filtro `business_type`. | Lista cronológica de `{create_date, amount, operation_type, category, business_type}`. |
| `/api/metrics/summary` | Fechas, `category`, `operation_type`, `business_type` opcionales; `group_by`. | Lista cronológica de `{period, income, outcome, net}`. |
| `/api/metrics/alerts` | Fechas, `business_type` opcionales; `group_by`; `threshold=0.3`, mínimo 0. | Lista cronológica de `{period, outcome_total, baseline_average, increase_ratio}`. |
| `/api/metrics/categories/top` | Fechas, `business_type` opcionales; `operation_type=outcome`; `limit=5`, entre 1 y 20. | Lista descendente por importe de `{category, operation_type, total_amount}`. Para esta funcionalidad enviar `operation_type=income` por separado para B2B y B2C. |
| `/api/metrics/b2b`, `/api/metrics/b2c` | Fechas, `category`, `operation_type` opcionales. | Movimientos con la misma estructura de `/api/metrics`, limitados al grupo de la ruta. Alternativa si se necesitan registros individuales. |
| `/api/metrics/comparison` | Fechas obligatorias; `business_type` opcional. | `{current_period, previous_period, delta_abs, delta_pct}`; compara beneficio neto con el período anterior de igual duración, no ingresos entre grupos. `delta_pct` puede ser `null`. |
| `/health` | Ninguno. | `{status: "ok"}`; no participa en las tres funcionalidades. |

`period` usa `YYYY-MM-DD`, `YYYY-Www` (semana ISO) o `YYYY-MM`.
La API declara validación de tipos, enumeraciones y límites mediante FastAPI;
la respuesta 422 y su esquema concreto quedan pendientes de contraste con OpenAPI.

## Casos límite de aceptación

| Funcionalidad | Casos mínimos |
| --- | --- |
| Fechas | Inicio igual a fin incluye ese día; solo inicio incluye fechas posteriores o iguales; solo fin incluye fechas anteriores o iguales; inicio posterior a fin se bloquea en UI; rango válido sin movimientos muestra vacío. |
| Alertas | Un único período no genera alerta; incremento igual al umbral no genera alerta; media anterior cero tampoco; `[]` muestra el mensaje vacío explícito. |
| B2B/B2C | Top B2B vacío muestra su mensaje sin ocultar el top B2C y viceversa; ambos tops vacíos muestran ambos mensajes; menos de cinco categorías no se rellena; períodos distintos se alinean con cero solo en presentación. |

## Pendientes y alcance

- Contraste con `http://localhost:8000/docs` o `/openapi.json`: no hay esquema
  guardado ni se ha consultado el servicio; los contratos proceden del código.
- Confirmar rango inicial propuesto (todo el disponible), referencia histórica
  de alertas dentro del filtro y moneda. No asumir persistencia del umbral.
- Las props de UI son propuestas. Las respuestas agregadas no están declaradas
  en los tipos actuales del frontend; su integración futura no forma parte de esta tarea.
- El ejercicio pide explorar y documentar el proyecto: estas especificaciones
  no afirman que las funcionalidades estén implementadas o verificadas en ejecución.