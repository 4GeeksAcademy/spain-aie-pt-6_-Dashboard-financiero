# Componentes propuestos

Especificación, no implementación. Las props siguientes son contratos de UI
propuestos, no campos nuevos de la API ni archivos de tipos adicionales.
Conservar los componentes existentes del dashboard y su sistema de estilos.

| Componente propuesto | Props | Responsabilidad |
| --- | --- | --- |
| `DateFilters` | `startDate`, `endDate` (fecha o sin valor), `minDate`, `maxDate`, `loading`, `error`, `onChange(startDate, endDate)`, `onReset()` | Dos controles de fecha; mostrar el rango disponible; impedir aplicar un rango invertido o fuera de límites. |
| `AlertsSection` | `threshold` (número), `groupBy` (`day`, `week`, `month`), `onThresholdChange(value)`, `onGroupByChange(value)`, `alerts`, `loading`, `error` | Control numérico del umbral, selector de agrupación y tabla de alertas. Umbral inicial `0.3`. |
| `AlertsTable` | `alerts`, `loading`, `error` | Columnas correspondientes a `period`, `outcome_total`, `baseline_average`, `increase_ratio`; este último se muestra como porcentaje. |
| `B2BB2CComparison` | `b2bSummary`, `b2cSummary`, `b2bTop`, `b2cTop`, `loading`, `error` | Dos series de ingresos, totales derivados y top de categorías independiente para cada grupo. |

`alerts`, las dos series y los dos tops conservan exactamente las estructuras
de respuesta descritas en [README.md](README.md). Los callbacks no realizan
cálculos financieros ni definen transporte HTTP.

## Renderizado condicional

- Mientras cargan las facetas, deshabilitar los controles de fecha; si fallan,
  mostrar el error sin inventar límites.
- Para cada sección: carga con skeleton; error con mensaje; éxito con contenido
  o estado vacío. No mostrar datos anteriores como si pertenecieran al nuevo rango.
- Alertas sin resultados: mostrar **Sin alertas para el rango y umbral seleccionados**.
- Comparativa: mantener visibles ambos grupos; mostrar **Sin ingresos** o
  **Sin categorías de ingresos** en el grupo vacío, sin ocultar el otro.
- Un cero numérico no equivale a carga ni a error. Los períodos ausentes se
  alinean con cero solo para representar las series, sin alterar la respuesta API.

Reutilizar la lógica financiera y de formato existente fuera de los componentes.
Los nombres de componentes y props son propuestas; su integración queda pendiente.