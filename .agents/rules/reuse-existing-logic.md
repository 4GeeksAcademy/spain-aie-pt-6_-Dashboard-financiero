# Reutilización de lógica financiera y filtros existentes

**Cuándo se aplica:** Antes de escribir cualquier función que filtre, agregue, calcule o formatee datos financieros, ya sea en el frontend o en el backend.

## Regla

El proyecto ya incluye funciones de propósito específico para procesar datos financieros. No las dupliques.

### Frontend — Funciones disponibles en `frontend/src/lib/financial-utils.ts`

| Función | Entrada | Salida | Línea |
|---|---|---|---|
| `computeKPIs(movements)` | `FinancialMovement[]` | `KPIMetrics` (totalIncome, totalOutcome, profit, profitPercent) | 19 |
| `computeMonthlyData(movements)` | `FinancialMovement[]` | `MonthlyDataPoint[]` agregado por mes | 32 |
| `formatCurrency(value)` | `number` | cadena con formato USD (ej. "$1,235") | 59 |
| `formatPercent(value)` | `number` | cadena con formato porcentaje (ej. "15.6%") | 69 |

Estas funciones se importan y usan desde `frontend/src/App.tsx` (líneas 27-28). Cualquier nueva lógica de cálculo financiero debe añadirse a este archivo, no dentro de un componente React.

### Backend — Filtros disponibles como parámetros de query

Los siguientes filtros ya están implementados en los endpoints de `backend/app/routes.py`. No los implementes de nuevo en JavaScript:

- `start_date` / `end_date` — filtro por rango de fechas
- `category` — filtro por categoría (suppliers, sales, operational, administrative, others)
- `operation_type` — filtro por tipo (income, outcome)
- `business_type` — filtro por tipo de negocio (B2B, B2C) en los endpoints `/api/metrics/b2b` y `/api/metrics/b2c`

La función `filter_movements()` (líneas 121-139) aplica estos filtros en el backend.

### No implementar lógica financiera dentro de componentes

Los componentes React (`kpi-row.tsx`, `App.tsx`, etc.) deben limitarse a renderizar datos ya procesados. La lógica de negocio va en `financial-utils.ts`.

## Qué hacer

1. Antes de escribir una función que procese datos financieros, busca si ya existe en `financial-utils.ts` o como parámetro de un endpoint.
2. Si la funcionalidad no existe, añádela al archivo correspondiente (`financial-utils.ts` para frontend, nuevo endpoint o función en `routes.py` para backend).
3. Después de cualquier cambio, ejecuta los tests:
   - Frontend: `npm test` (desde `frontend/`)
   - Backend: `pytest` (desde `backend/`)

## Archivos de referencia

- `frontend/src/lib/financial-utils.ts` — funciones de cálculo y formato
- `frontend/src/App.tsx` — punto de uso de las funciones
- `backend/app/routes.py` — función `filter_movements()` y parámetros de endpoints