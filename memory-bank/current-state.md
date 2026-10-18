# Estado actual del proyecto — Financial Metrics Dashboard

## Estado general

El repositorio ha sido analizado en profundidad durante una sesión de onboarding. Se han documentado reglas para agentes de IA, se ha verificado el resumen del proyecto y se ha creado este banco de memoria. **No se ha modificado ningún archivo de código, configuración ni documentación preexistente.**

## Endpoints de la API (9 endpoints)

Todos en `backend/app/routes.py`:

| # | Endpoint | Línea | Descripción |
|---|---|---|---|
| 1 | `GET /health` | 243 | Health check simple |
| 2 | `GET /api/metrics` | 248 | Lista de movimientos (con filtros opcionales: start_date, end_date, category, operation_type) |
| 3 | `GET /api/metrics/facets` | 262 | Facetas disponibles (tipos, categorías, rango de fechas) |
| 4 | `GET /api/metrics/summary` | 268 | Resumen agrupado por day/week/month (con filtros) |
| 5 | `GET /api/metrics/categories/top` | 287 | Top N categorías por importe (por operation_type) |
| 6 | `GET /api/metrics/comparison` | 305 | Comparativa entre dos períodos (requiere start_date y end_date) |
| 7 | `GET /api/metrics/alerts` | 342 | Alertas de gasto anómalo respecto a la media histórica |
| 8 | `GET /api/metrics/b2b` | 362 | Movimientos filtrados a B2B |
| 9 | `GET /api/metrics/b2c` | 378 | Movimientos filtrados a B2C |

Todos los endpoints usan `generate_mock_movements(seed=42)` y aceptan diversos parámetros de filtrado vía Query.

## Reglas existentes en `.agents/rules/`

| Archivo | Propósito |
|---|---|
| `.agents/rules/api-contract-sync.md` | Mantener sincronizados los modelos Pydantic (backend) con las interfaces TypeScript (frontend) |
| `.agents/rules/reuse-existing-logic.md` | Reutilizar `financial-utils.ts` para cálculos y los filtros ya implementados como query params en los endpoints |
| `.agents/rules/deterministic-data-integrity.md` | Preservar `seed=42`, los 360 movimientos y demás parámetros de generación de datos mock |
| `.agents/rules/infrastructure-preservation.md` | Mantener el proxy de Vite, puertos, volúmenes Docker y variable `VITE_API_BASE_URL` |
| `.agents/rules/css-variable-system.md` | Usar las variables CSS personalizadas en espacio de color `oklch()` definidas en `index.css`, no valores fijos de Tailwind |

## Verificaciones realizadas

Documentadas en `verification.md` (raíz del repositorio):

| Afirmación | Resultado |
|---|---|
| Backend genera 360 movimientos con seed=42 | ✅ VERIFICADO |
| Frontend obtiene datos por API y no usa mock-data.ts | ✅ VERIFICADO (mock-data.ts existe pero no es importado) |
| Existen 9 endpoints en routes.py | ✅ VERIFICADO |
| Vite proxy dirige /api → backend:8000 | ✅ VERIFICADO |
| **Conteo de tests: 8 backend + 5 frontend** | ❌ **INCORRECTO** → la cifra real es **15 backend + 5 frontend = 20 tests** |

### Corrección del recuento de tests

El resumen inicial afirmaba 8 tests de backend. La cifra real es **15 tests** en `backend/tests/test_routes.py`. Adicionalmente hay **5 tests** en `frontend/src/lib/financial-utils.test.ts`. Total: **20 tests declarados**.

**No se ha ejecutado ninguna de las dos baterías**, por lo que no se puede afirmar que los tests pasen.

## Limitaciones conocidas (respaldadas por evidencia)

| Limitación | Evidencia |
|---|---|
| Sin base de datos — los datos se generan en memoria en cada petición | `backend/app/routes.py` función `generate_mock_movements()` |
| `frontend/src/lib/mock-data.ts` no se utiliza en tiempo de ejecución | Búsqueda de imports no encontró ninguna referencia al archivo |
| No existe `frontend/.env.example` a pesar de mencionarse en README.md | El directorio `frontend/` no contiene dicho archivo |
| Sin tests de integración, E2E ni de carga | Solo existen tests unitarios (pytest + vitest) |
| Sin autenticación ni control de acceso | No hay middleware, dependencias ni rutas de login |
| `components.json` de shadcn/ui presente pero no se usa activamente | No se encontraron imports de shadcn/cli en el proyecto |

## Tareas pendientes

- [ ] Ejecutar los tests de backend (`pytest` desde `backend/`) para confirmar que los 15 tests pasan.
- [ ] Ejecutar los tests de frontend (`npm test` desde `frontend/`) para confirmar que los 5 tests pasan.
- [ ] Decidir si `mock-data.ts` debe eliminarse o integrarse como alternativa offline.
- [ ] Decidir si `components.json` (shadcn/ui) debe eliminarse o integrarse.
- [ ] Crear `frontend/.env.example` si se desea mantener el workflow de `VITE_API_BASE_URL` documentado en el README.
- [ ] Ampliar la cobertura de tests si se añaden nuevas funcionalidades.

**Nota:** Ninguna de estas tareas se ha ejecutado. Se listan como elementos detectados durante el análisis.