---
name: financial-dashboard-workflow
description: Use when implementing or changing financial dashboard views that combine API data, date filters, alerts, or B2B/B2C comparisons.
---

# Financial Dashboard Workflow

## Objetivo y cuándo usarla

Integrar una funcionalidad financiera de extremo a extremo sin asumir que los
endpoints comparten parámetros ni que un resultado vacío equivale a un error.
Úsala al añadir o modificar filtros, agregados, alertas o comparativas del
dashboard. Para accesibilidad, pruebas React o rendimiento, usa las skills
comunitarias correspondientes.

## Entradas necesarias

- El requisito y su especificación aplicable en `frontend/specs/`.
- El contrato real del endpoint en `backend/app/routes.py` y sus modelos.
- Los tipos y cálculos existentes en `frontend/src/lib/financial-types.ts` y
  `frontend/src/lib/financial-utils.ts`.
- Las reglas relevantes de `.agents/rules/`, en particular sincronización de
  contratos y reutilización de lógica.

## Procedimiento

1. Localiza la especificación más cercana. Si no cubre el caso, confirma los
   parámetros, campos de respuesta y semántica de vacío en la ruta del backend;
   no deduzcas el contrato por el nombre de un endpoint.
2. Para cada consulta, registra qué filtros admite. Comparte el mismo rango de
   fechas entre secciones solo cuando cada endpoint lo soporte; omite extremos
   opcionales ausentes, sin enviarlos como cadenas vacías.
3. Conserva las respuestas del API y reutiliza tipos y utilidades existentes.
   En comparativas B2B/B2C, consulta cada grupo y cada ranking por separado;
   alinea períodos ausentes solo para presentación. No recalcules en UI
   resultados que el backend ya entrega, como las alertas.
4. Mantén diferenciados carga, error, éxito vacío y éxito con datos por sección.
   Al cambiar filtros, no presentes datos anteriores como si correspondieran al
   nuevo rango. Respeta los casos límite de aceptación de la especificación.
5. Si cambia el contrato, aplica `.agents/rules/api-contract-sync.md`. Ejecuta
   las comprobaciones enfocadas de las capas modificadas y comunica cualquier
   verificación pendiente.

## Resultado esperado

Una vista financiera conectada a consultas compatibles con sus contratos,
coherente con las reglas del proyecto y con estados y agregados que representan
fielmente las respuestas, incluso cuando no hay resultados.

## Criterios de aceptación

- Cada parámetro enviado existe en la firma del endpoint correspondiente y cada
  campo representado existe en su respuesta documentada.
- Los filtros compartidos producen resultados del mismo rango en todas las
  secciones compatibles; las consultas por grupo no se sustituyen por resultados
  combinados.
- `[]`, error y carga se muestran como estados distintos; no se inventan filas,
  categorías ni períodos fuera de los datos disponibles.
- Se cumplen los casos límite de la especificación seleccionada y se informa
  qué comprobaciones se ejecutaron.