# Integridad de los datos mock deterministas

**Cuándo se aplica:** Al modificar cualquier aspecto de la generación de datos simulados (semilla, cantidad de movimientos, rangos de montos, probabilidades, categorías) o al añadir nuevos campos al modelo `FinancialMovement`.

## Regla

El backend genera datos financieros simulados de forma determinista para que cada ejecución produzca los mismos resultados. Varias partes del proyecto dependen de estos valores concretos.

### Parámetros actuales de generación

| Parámetro | Valor | Localización |
|---|---|---|
| Semilla | `seed=42` | `backend/app/routes.py` líneas 250, 264, 270, 296, 314, 346, 366, 382 |
| Total movimientos | 360 (12 meses × 30 movimientos) | `backend/app/routes.py` líneas 99-103 |
| Meses simulados | 12 (enero a diciembre del año actual) | `backend/app/routes.py` línea 99 |
| Rango montos income | 800 – 12 000 | `backend/app/routes.py` línea 76 |
| Rango montos outcome | 500 – 9 000 | `backend/app/routes.py` línea 79 |
| Probabilidad income | 45 % – 70 % según el mes | `backend/app/routes.py` línea 100 |
| Proporción B2B | ~55 % | `backend/app/routes.py` línea 65 |
| Días del mes | 1 – 28 | `backend/app/routes.py` línea 63 |

### Lo que verifica el test existente

El test `test_generate_mock_movements_returns_full_year_sorted_data` en `backend/tests/test_routes.py` (línea 12) comprueba que:

- `len(movements) == 360`
- Los movimientos están ordenados cronológicamente

Si modificas los parámetros de generación, este test fallará a menos que lo actualices.

## Qué hacer

1. **No cambiar `seed=42`** en ninguna llamada a `generate_mock_movements()`. Es la semilla que garantiza datos reproducibles en todas las ejecuciones y tests.

2. **Si modificas la cantidad de movimientos** (30 por mes o 12 meses), actualiza el test en `backend/tests/test_routes.py` línea 14 (`assert len(movements) == 360`).

3. **Si modificas los rangos de montos, probabilidades o categorías**, comprueba que ningún test dependa de valores numéricos concretos que hayan cambiado. Si es así, actualízalos.

4. **Si añades un nuevo campo a `FinancialMovement`**, actualiza:
   - El modelo Pydantic en `backend/app/routes.py` (clase `FinancialMovement`, línea 22)
   - La interfaz TypeScript en `frontend/src/lib/financial-types.ts` (interfaz `FinancialMovement`, línea 6)
   - La función `_build_movement()` en `backend/app/routes.py` (línea 57) para poblar el nuevo campo
   - Los tests relevantes

5. **Ejecuta `pytest`** (desde `backend/`) para verificar que todos los tests siguen pasando.

## Archivos de referencia

- `backend/app/routes.py` — función `generate_mock_movements()` (línea 94), función `_build_movement()` (línea 57)
- `backend/tests/test_routes.py` — test de generación (línea 12), tests de filtros y endpoints