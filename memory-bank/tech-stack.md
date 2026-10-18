# Tech Stack — Financial Metrics Dashboard

## Frontend

| Tecnología | Versión | Propósito | Archivo de referencia |
|---|---|---|---|
| React | ^19.2.4 | Framework UI con JSX runtime | `frontend/package.json` |
| TypeScript | ~6.0.2 | Tipado estático | `frontend/tsconfig.json` |
| Vite | ^8.0.4 | Servidor de desarrollo y build | `frontend/vite.config.ts` |
| Tailwind CSS v4 | ^4.2.2 | Estilos utility-first con variables `oklch()` | `frontend/src/index.css` |
| Recharts | ^3.8.1 | Gráficos de líneas (ingresos/gastos, margen) | `frontend/package.json` |
| Lucide React | ^1.8.0 | Iconos (TrendingUp, DollarSign, etc.) | `frontend/package.json` |
| Vitest | ^4.1.4 | Tests unitarios | `frontend/package.json` |
| ESLint | ^9.39.4 | Linter con config plana | `frontend/eslint.config.js` |
| shadcn/ui | — | Sistema de componentes (Card, Skeleton) | `frontend/components.json` |

**Alias de importación:** `@` → `./src` (configurado en `tsconfig.app.json` y `vite.config.ts`).

## Backend

| Tecnología | Versión | Propósito | Archivo de referencia |
|---|---|---|---|
| Python | 3.13 | Lenguaje runtime | `backend/Dockerfile` |
| FastAPI | — | Framework ASGI para API REST | `backend/app/main.py` |
| Uvicorn | — | Servidor ASGI con flag `--reload` | `backend/Dockerfile` |
| Pydantic | — | Modelos de datos y validación | `backend/app/routes.py` |
| debugpy | — | Depurador remoto (puerto 5678) | `backend/Dockerfile` |
| pytest | — | Tests unitarios | `backend/requirements.txt` |
| pytest-cov | — | Cobertura de tests | `backend/requirements.txt` |
| httpx | — | Cliente HTTP para tests | `backend/requirements.txt` |

## Comunicación entre servicios

```
Frontend (Vite :5173) → proxy /api → Backend (FastAPI :8000)
```

- **Proxy de Vite** (`frontend/vite.config.ts` líneas 10-15): redirige todas las peticiones `/api` a `http://backend:8000`. El nombre `backend` se resuelve por DNS interno de Docker Compose.
- **Variable de entorno alternativa:** `VITE_API_BASE_URL` (en `frontend/src/App.tsx` línea 13). Si se define, se antepone a las rutas API. Si no, se usa cadena vacía y el proxy gestiona el enrutamiento.
- **CORS:** El backend también permite CORS para todos los orígenes (`backend/app/main.py`), lo que permite acceder directamente desde fuera de Docker.

## Configuración de Docker Compose

Archivo: `docker-compose.yml`

| Servicio | Puerto(s) | Dockerfile | Volúmenes |
|---|---|---|---|
| frontend | 5173 | `frontend/Dockerfile` | `./frontend:/app` + anonymous `/app/node_modules` |
| backend | 8000 (API) + 5678 (debug) | `backend/Dockerfile` | `./backend:/app` |

- `depends_on`: frontend → backend.
- Los bind mounts permiten desarrollo en caliente (hot reload).
- El volumen anónimo `/app/node_modules` evita sobrescribir los módulos del contenedor con los del host.

### Cómo ejecutar

```bash
# Desde la raíz del repositorio
docker compose up --build
```

- Frontend: http://localhost:5173
- Backend: http://localhost:8000
- Documentación API: http://localhost:8000/docs

## Tests disponibles

### Backend (15 tests)

```bash
# Desde el directorio backend/
pytest
```

- Archivo: `backend/tests/test_routes.py`
- Framework: pytest con `TestClient` de FastAPI
- 15 tests que cubren: generación de datos, filtros por fecha/categoría/tipo, endpoints (`/health`, `/api/metrics`, `/api/metrics/b2b`, `/api/metrics/b2c`, etc.)

### Frontend (5 tests)

```bash
# Desde el directorio frontend/
npm test            # vitest run
npm run test:watch  # vitest (modo watch)
npm run test:coverage  # vitest run --coverage
```

- Archivo: `frontend/src/lib/financial-utils.test.ts`
- Framework: Vitest
- 5 tests que cubren: `computeKPIs()`, `computeMonthlyData()`, `formatCurrency()`, `formatPercent()`

**Nota:** Estos son los tests declarados en el código. No se ha confirmado que todos pasen porque no se han ejecutado en esta sesión.