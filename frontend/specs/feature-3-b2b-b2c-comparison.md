# 3. Ingresos B2B frente a B2C

## Consultas necesarias

Usar las mismas fechas inclusivas `YYYY-MM-DD` y agrupación para ambos grupos.
Realizar cuatro consultas lógicas independientes:

| Endpoint | Parámetros específicos | Respuesta |
| --- | --- | --- |
| `/api/metrics/summary` | `business_type=B2B`, `operation_type=income`, `group_by=month` | Lista de `{period, income, outcome, net}`. |
| `/api/metrics/summary` | `business_type=B2C`, `operation_type=income`, `group_by=month` | Misma estructura. |
| `/api/metrics/categories/top` | `business_type=B2B`, `operation_type=income`, `limit=5` | Lista de `{category, operation_type, total_amount}`. |
| `/api/metrics/categories/top` | `business_type=B2C`, `operation_type=income`, `limit=5` | Misma estructura; ranking propio de B2C. |

Las cuatro admiten `start_date` y `end_date` opcionales. `group_by` solo se envía
al resumen. No obtener un top combinado para después repartirlo entre grupos.
No usar `/api/metrics/comparison`: compara beneficio neto entre dos períodos.

## Representación

- Comparar el campo `income` por período, con una serie identificada por grupo.
  Con este filtro, `outcome` es cero y `net` coincide con `income`.
- El total de cada grupo se deriva sumando `income` del resumen completo,
  nunca sumando únicamente las categorías del top.
- Alinear las series por la unión ordenada de sus períodos; un período ausente
  en un grupo se representa como cero. No inventar períodos fuera de esa unión.
- Presentar dos rankings separados, ordenados por `total_amount` descendente.
  El top corresponde al rango completo, no a cada período de la serie.
- El backend devuelve hasta cinco categorías, no exactamente cinco. Los ingresos
  simulados solo usan `sales` y `others`; no rellenar con categorías ficticias.

## Estados y límites

- Un grupo sin ingresos conserva su espacio y muestra un estado vacío explícito;
  el grupo con datos sigue visible. Si ambos están vacíos, indicarlo para ambos.
- Si el top B2B devuelve `[]`, mostrar **Sin categorías de ingresos para B2B**
  en su ranking; si el top B2C devuelve `[]`, mostrar **Sin categorías de ingresos
  para B2C** en el suyo. Evaluar cada top de forma independiente; si ambos están
  vacíos, mostrar ambos mensajes sin filas ficticias.
- Distinguir carga, error y éxito vacío; no representar un fallo como ingresos cero.
- Los empates del top no tienen desempate contractual; no depender de su orden.
- No se exige diferencia porcentual entre grupos; evitar divisiones por cero
  al interpretar resultados de un grupo sin ingresos.

Pendiente: confirmar el diseño visual de la comparativa y la moneda, ausente
del contrato API. El top por separado para cada grupo forma parte del requisito.