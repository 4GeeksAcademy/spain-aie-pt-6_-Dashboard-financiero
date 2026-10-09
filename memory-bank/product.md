# Producto — Financial Metrics Dashboard

## Qué es y para qué sirve

Es un panel web (dashboard) de métricas financieras que muestra ingresos, gastos, beneficio neto y margen de beneficio a partir de datos financieros simulados. Está diseñado como proyecto educativo de 4Geeks Academy para que agentes de IA analicen un código real, documenten reglas y creen un banco de memoria.

- **Propósito educativo:** Servir como repositorio de práctica para ingenieros de IA, no como herramienta de producción.
- **Frontend:** React 19 + TypeScript ~6.0 + Vite 8.
- **Backend:** Python 3.13 + FastAPI.
- **Infraestructura:** Docker Compose con dos servicios.

## Funcionalidades actuales del dashboard

El dashboard (implementado en `frontend/src/App.tsx`) muestra:

1. **Cabecera** (`DashboardHeader`): período "2024 - Full Year".
2. **4 tarjetas KPI** (`KPIRow` + `KPICard`):
   - Total Income — ingresos totales acumulados.
   - Total Outcome — gastos totales acumulados.
   - Profit — beneficio neto (ingresos − gastos).
   - Profit Margin — beneficio como porcentaje de los ingresos.
3. **Gráfico de líneas Ingresos vs Gastos** (`IncomeOutcomeChart`): evolución mensual usando Recharts.
4. **Gráfico de margen de beneficio** (`ProfitPercentChart`): porcentaje mensual con línea de referencia en 0%.
5. **Esqueleto (skeleton) de carga**: todos los componentes muestran un placeholder animado mientras `loading === true`.
6. **Manejo de errores**: si la API no responde, se muestra un mensaje en rojo dentro del dashboard.

No hay autenticación, paginación, exportación de datos, ni interactividad avanzada (los gráficos son estáticos).

## Datos que utiliza

- **360 movimientos financieros simulados**, generados por `generate_mock_movements(seed=42)` en `backend/app/routes.py` (líneas 94-104). La semilla `42` garantiza que cada ejecución produzca los mismos datos.
- **Estructura de cada movimiento:** fecha, importe, tipo (income/outcome), categoría (suppliers, sales, operational, administrative, others) y tipo de negocio (B2B/B2C).
- **Cálculo de KPIs:** se realiza en el frontend mediante `computeKPIs()` y `computeMonthlyData()` en `frontend/src/lib/financial-utils.ts`. El backend envía los movimientos en bruto; el frontend los agrega.
- **No hay base de datos.** Todos los datos se generan en memoria en cada petición.

## Limitaciones

- **Datos simulados, no reales.** Cualquier inferencia sobre tendencias financieras es ficticia.
- **Sin persistencia.** No hay base de datos, cache ni almacenamiento. Al reiniciar el backend se regenera todo.
- **`mock-data.ts` sin usar.** El archivo `frontend/src/lib/mock-data.ts` contiene datos estáticos de 2024 pero no es importado por ningún archivo del proyecto (verificado en `verification.md`).
- **No apto para producción.** No tiene logging estructurado, autenticación, rate limiting, ni monitorización.
- **Sin `.env.example`.** El README menciona `frontend/.env.example` para `VITE_API_BASE_URL`, pero no existe en el repositorio.

## Aspectos que NO deben darse por implementados sin evidencia

- No asumas que existe autenticación de usuarios.
- No asumas que los datos provienen de una base de datos real o de una API externa.
- No asumas que `mock-data.ts` se utiliza en tiempo de ejecución (está verificado que no se importa).
- No asumas que hay tests de integración, E2E o de carga.
- No asumas que el dashboard soporta múltiples usuarios, períodos seleccionables o filtros desde la interfaz.
- No asumas que los tests pasan si no se han ejecutado explícitamente.