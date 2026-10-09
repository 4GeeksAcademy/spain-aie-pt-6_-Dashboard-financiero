# Sincronización de contratos API

**Cuándo se aplica:** Siempre que se modifique un modelo de datos, un tipo literal, una ruta de endpoint o la estructura de una respuesta de la API.

## Regla

Los modelos y tipos de datos están definidos en dos lugares que deben modificarse siempre **a la vez**:

| Concepto | Backend (Pydantic) | Frontend (TypeScript) |
|---|---|---|
| `FinancialMovement` | `backend/app/routes.py` clase `FinancialMovement(BaseModel)` (línea 22) | `frontend/src/lib/financial-types.ts` interfaz `FinancialMovement` (línea 6) |
| `OperationType` | `backend/app/routes.py` `Literal["income", "outcome"]` (línea 9) | `frontend/src/lib/financial-types.ts` `type OperationType` (línea 1) |
| `Category` | `backend/app/routes.py` `Literal["suppliers", "sales", "operational", "administrative", "others"]` (líneas 10-11) | `frontend/src/lib/financial-types.ts` `type Category` (líneas 2-3) |
| `BusinessType` | `backend/app/routes.py` `Literal["B2B", "B2C"]` (línea 12) | `frontend/src/lib/financial-types.ts` `type BusinessType` (línea 4) |
| `GroupBy` | `backend/app/routes.py` `Literal["day", "week", "month"]` (línea 13) | — (solo backend) |

## Qué hacer

1. **Si añades, eliminas o renombras un campo** en una clase Pydantic del backend, replica el cambio en la interfaz TypeScript equivalente del frontend.

2. **Si cambias la ruta de un endpoint**, actualiza también la URL en `frontend/src/App.tsx` (línea 17: `fetch(\`${API_BASE_URL}/api/metrics\`)`) y en cualquier otro archivo que llame a esa ruta.

3. **Si añades un nuevo endpoint** con el prefijo `/api`, el proxy de Vite lo cubrirá automáticamente (configurado en `frontend/vite.config.ts` líneas 11-15). No necesitas configuración adicional.

4. **Ejecuta las pruebas** después de los cambios:
   - Backend: `pytest` (desde `backend/`)
   - Frontend: `npm test` (desde `frontend/`)
   - Si modificaste ambos lados, ejecuta ambas baterías.

## Archivos de referencia

- `backend/app/routes.py` — modelos Pydantic, type aliases y endpoints
- `frontend/src/lib/financial-types.ts` — interfaces y tipos TypeScript
- `frontend/src/App.tsx` — llamada `fetch` a la API
- `frontend/vite.config.ts` — proxy de Vite para `/api`