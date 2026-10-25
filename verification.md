# Verificación del resumen del proyecto — Financial Metrics Dashboard

## 1. Objetivo

Este documento contrasta las afirmaciones del resumen del proyecto con el código real del repositorio. Cada afirmación se clasifica como **VERIFICADO**, **INCORRECTO** o **SIN VERIFICAR**, y se respalda con la ruta del archivo y las líneas de código que la sustentan.

---

## 2. Afirmaciones verificadas

### 2.1 El backend genera 360 movimientos con `seed=42`

**VERIFICADO**

- La función `generate_mock_movements` itera 12 meses y dentro de cada mes genera 30 movimientos (12 × 30 = 360):
  - **Archivo:** `backend/app/routes.py`, líneas 94-104
  - **Fragmento clave:**
    ```python
    def generate_mock_movements(seed: int | None = None) -> list[FinancialMovement]:
        if seed is not None:
            random.seed(seed)
        today = date.today()
        movements: list[FinancialMovement] = []
        for month in range(1, 13):          # 12 meses
            income_probability = random.uniform(0.45, 0.7)
            for _ in range(30):             # 30 movimientos por mes
                movements.append(_build_movement(month, income_probability, today))
        movements.sort(key=lambda item: item.create_date)
        return movements
    ```

- Todos los endpoints llaman a `generate_mock_movements(seed=42)`, por ejemplo:
  - **Archivo:** `backend/app/routes.py`, línea 250 (`get_metrics` → `generate_mock_movements(seed=42)`)
  - Análogamente en los otros 8 endpoints.

- El test específico confirma la cantidad exacta:
  - **Archivo:** `backend/tests/test_routes.py`, líneas 12-17
  - **Fragmento clave:**
    ```python
    def test_generate_mock_movements_returns_full_year_sorted_data():
        movements = generate_mock_movements(seed=42)
        assert len(movements) == 360
        assert movements == sorted(movements, key=lambda item: item.create_date)
    ```

---

### 2.2 El frontend obtiene los datos mediante la API y no importa `mock-data.ts`

**VERIFICADO**

- El frontend realiza una petición `fetch` a la ruta `/api/metrics`:
  - **Archivo:** `frontend/src/App.tsx`, líneas 13-20
  - **Fragmento clave:**
    ```typescript
    const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "";

    async function fetchFinancialData(): Promise<FinancialMovement[]> {
      const response = await fetch(`${API_BASE_URL}/api/metrics`);
      if (!response.ok) {
        throw new Error(`Failed to fetch financial data: ${response.status}`);
      }
      return response.json();
    }
    ```

- El archivo `frontend/src/lib/mock-data.ts` existe en disco pero **no es importado por ningún archivo** del proyecto. Una búsqueda con `import.*mock-data` en todo el repositorio no encontró resultados.

---

### 2.3 Existen 9 endpoints en `backend/routes.py`

**VERIFICADO**

- Se contaron **9 decoradores `@router.get`** en `backend/app/routes.py`:

| # | Endpoint | Línea |
|---|---|---|
| 1 | `GET /health` | 243 |
| 2 | `GET /api/metrics` | 248 |
| 3 | `GET /api/metrics/facets` | 262 |
| 4 | `GET /api/metrics/summary` | 268 |
| 5 | `GET /api/metrics/categories/top` | 287 |
| 6 | `GET /api/metrics/comparison` | 305 |
| 7 | `GET /api/metrics/alerts` | 342 |
| 8 | `GET /api/metrics/b2b` | 362 |
| 9 | `GET /api/metrics/b2c` | 378 |

- **Archivo:** `backend/app/routes.py`. Resultado de la búsqueda `@router\.get`.

---

### 2.4 Vite dirige las peticiones `/api` a `http://backend:8000`

**VERIFICADO**

- **Archivo:** `frontend/vite.config.ts`, líneas 10-15
- **Fragmento clave:**
    ```typescript
    server: {
        host: "0.0.0.0",
        proxy: {
            "/api": {
                target: "http://backend:8000",
                changeOrigin: true,
            },
        },
    },
    ```
- Esto redirige todas las peticiones cuyo path comience por `/api` hacia el servicio `backend` en el puerto `8000`, que se resuelve por DNS interno de Docker Compose.

---

## 3. Afirmación corregida

### 3.1 Conteo de tests

**El resumen inicial indicaba 8 tests de backend, pero la cifra real es 15.**

- **Tests de backend (`backend/tests/test_routes.py`): 15 tests**
  - 15 funciones `def test_` encontradas con búsqueda en el archivo:
    1. `test_generate_mock_movements_returns_full_year_sorted_data` (línea 12)
    2. `test_filter_movements_by_date_includes_range_edges` (línea 19)
    3. `test_health_endpoint_returns_ok` (línea 29)
    4. `test_metrics_endpoint_respects_date_filters` (línea 36)
    5. `test_b2b_endpoint_only_returns_b2b_records` (línea 52)
    6. `test_b2c_endpoint_only_returns_b2c_records` (línea 62)
    7. `test_metrics_endpoint_filters_by_category` (línea 72)
    8. `test_metrics_endpoint_filters_by_operation_type` (línea 81)
    9. `test_b2b_endpoint_combines_new_filters` (línea 90)
    10. `test_metrics_facets_returns_filter_options_and_date_range` (línea 104)
    11. `test_metrics_summary_by_month_returns_balances` (línea 121)
    12. `test_metrics_summary_by_week_honors_business_type_filter` (línea 133)
    13. `test_top_categories_returns_limited_sorted_categories` (línea 144)
    14. `test_metrics_comparison_returns_delta_fields` (línea 157)
    15. `test_metrics_alerts_returns_anomaly_candidates` (línea 173)

- **Tests de frontend (`frontend/src/lib/financial-utils.test.ts`): 6 tests** (declarados; no ejecutados)
  - 6 llamadas `it(...)`:
    1. `"calculates totals and profit values"` (línea 36)
    2. `"returns 0 profitPercent when there is no income"` (línea 47)
    3. `"returns chronological year-month points with aggregated totals"` (línea 64)
    4. `"returns a zero profit margin for a month with no income"` (línea 105)
    5. `"formats currency without decimals"` (línea 128)
    6. `"formats percent with one decimal"` (línea 132)

  - **Total: 21 tests declarados** (15 backend + 6 frontend).

---

## 4. Evidencias (resumen)

| Afirmación | Archivo | Líneas |
|---|---|---|
| Generación 360 movimientos con `seed=42` | `backend/app/routes.py` | 94-104 |
| Todos los endpoints usan `seed=42` | `backend/app/routes.py` | 250, 264, 270, 296, 314, 346, 366, 382 |
| Test confirma 360 movimientos | `backend/tests/test_routes.py` | 12-17 |
| Frontend obtiene datos vía API (`/api/metrics`) | `frontend/src/App.tsx` | 13-20 |
| `mock-data.ts` no se importa | — | Búsqueda `import.*mock-data` sin resultados |
| 9 endpoints en routes.py | `backend/app/routes.py` | 243, 248, 262, 268, 287, 305, 342, 362, 378 |
| Proxy Vite `/api` → `http://backend:8000` | `frontend/vite.config.ts` | 10-15 |
| 15 tests backend | `backend/tests/test_routes.py` | 12, 19, 29, 36, 52, 62, 72, 81, 90, 104, 121, 133, 144, 157, 173 |
| 6 tests frontend | `frontend/src/lib/financial-utils.test.ts` | 36, 47, 64, 105, 128, 132 |

---

## 5. Limitaciones

- **El conteo de tests refleja los tests definidos en el código, no su estado de ejecución.** Que existan 21 tests declarados no significa que todos se hayan ejecutado correctamente o que pasen en el entorno actual. Para verificarlo sería necesario ejecutar `pytest` en el backend y `npm test` en el frontend.
- Este documento verifica únicamente las afirmaciones del resumen. No cubre la totalidad del código ni posibles bugs o incoherencias no mencionadas en el resumen original.
- La ausencia de importación de `mock-data.ts` se ha verificado mediante búsqueda textual; no se ha analizado la posibilidad de carga dinámica o indirecta a través de otros mecanismos (p. ej., `require.context` o construcción dinámica de rutas), aunque no hay indicios de ello en el código.