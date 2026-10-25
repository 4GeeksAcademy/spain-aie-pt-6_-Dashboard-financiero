# 2. Tabla de alertas

## Contrato

`GET /api/metrics/alerts` admite:

- `threshold`: número mayor o igual que cero; por defecto `0.3`, sin máximo declarado.
- `group_by`: `day`, `week` o `month`; por defecto `month`.
- `start_date`, `end_date`: opcionales, inclusivas, formato `YYYY-MM-DD`.
- `business_type`: opcional, `B2B` o `B2C`; omitido incluye ambos.

Respuesta: lista cronológica de
`{period, outcome_total, baseline_average, increase_ratio}`.
No hay identificador, categoría, gravedad ni estado de gestión en la respuesta.

## Interacción y cálculo

- Control numérico con valor inicial `0.3` y etiqueta **Umbral de incremento**.
  Se edita una proporción: `0.3` equivale al 30 %. Rechazar valores vacíos,
  no numéricos o negativos antes de aplicar.
- Cambiar umbral, agrupación o fechas actualiza la tabla. El umbral es un
  parámetro de consulta; no hay endpoint para persistirlo.
- Cada período se compara con la media de gastos de todos los períodos
  anteriores presentes en el resumen del rango filtrado.
- Solo hay alerta si `(gasto - media) / media > threshold`, estrictamente.
  El primer período y los períodos con media histórica cero no generan alerta.
- No se incorporan períodos sin movimientos. El filtrado y la detección
  corresponden al backend; no recalcular alertas en componentes.

## Tabla y estados

Columnas: período, gasto total, media de referencia e incremento.
Mostrar `increase_ratio` como porcentaje; conservar los campos API originales.
`period` es `YYYY-MM-DD`, `YYYY-Www` (semana ISO) o `YYYY-MM`, según agrupación.

- Carga: skeleton; error: mensaje diferenciado, no estado vacío.
- Respuesta `[]`: **Sin alertas para el rango y umbral seleccionados**.
- Un solo período o un umbral alto pueden producir legítimamente `[]`.
- Un incremento exactamente igual al umbral no produce alerta.

Pendiente: confirmar si la referencia debe permanecer limitada al rango elegido;
es el comportamiento actual, no un histórico externo al filtro.