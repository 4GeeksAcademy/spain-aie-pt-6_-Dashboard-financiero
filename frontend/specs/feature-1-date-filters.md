# 1. Filtros de fechas

## Objetivo y datos

- Obtener `min_date` y `max_date` mediante `GET /api/metrics/facets`, sin parámetros.
  Son límites globales, no dependen de los filtros de la pantalla.
- Usar fechas `YYYY-MM-DD`, sin hora. `start_date` y `end_date` son inclusivas.
- Propuesta inicial: seleccionar todo el rango disponible, sin fijar un año.
  Los datos simulados abarcan los doce meses completos anteriores al mes actual.
- Aplicar el mismo rango a movimientos/KPIs, series, alertas y tops B2B/B2C.
  Los movimientos se obtienen de `/api/metrics`; las series, de `/api/metrics/summary`.

## Comportamiento

- Los dos extremos son opcionales en la API: omitir un extremo significa no
  limitar por ese lado; omitir ambos devuelve todo el conjunto disponible.
- Permitir aplicar solo `start_date`: incluir fechas mayores o iguales al inicio;
  solo `end_date`: incluir fechas menores o iguales al fin. Omitir el parámetro
  sin valor, sin enviarlo como cadena vacía ni completar el extremo automáticamente.
- Permitir cambiar cada extremo y aplicar solo un rango válido. Restablecer
  recupera `min_date` y `max_date`.
- Validar formato y pertenencia al rango disponible de cada fecha proporcionada;
  comprobar `start_date <= end_date` solo cuando existan ambos extremos.
  El backend no valida el orden del rango.
- Mantener los filtros de categoría o tipo de operación que ya estén activos
  únicamente en los endpoints que los admitan; no inventar parámetros comunes.
- Durante carga/error de facetas no usar fechas fijas como sustituto.

## Aceptación

- Un rango con el mismo inicio y fin incluye los movimientos de ese día.
- Un rango válido sin movimientos muestra estado vacío, no error; las respuestas
  de listas son `[]` y los totales derivados son cero.
- Un rango invertido se bloquea en UI; enviado directamente al backend produce
  listas vacías, no un error específico de rango.
- La selección actualiza todas las secciones con los mismos límites y no mezcla
  resultados de rangos distintos.

Pendiente: confirmar la propuesta de rango inicial y contrastar el contrato
con `/openapi.json` servido.