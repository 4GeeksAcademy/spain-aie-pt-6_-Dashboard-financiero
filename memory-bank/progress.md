# Progreso del ejercicio de skills

## Aplicación

- **accessibility:** `App.tsx` anuncia errores y carga; `card.tsx` renderiza los títulos como encabezados; los gráficos incluyen navegación accesible y tablas textuales; `index.css` define foco visible y respeta movimiento reducido.
- **vercel-react-best-practices:** `App.tsx` cancela la petición al desmontarse y deriva los KPI y datos mensuales durante el render, sin memoización innecesaria.
- **react-testing-best-practices:** `financial-utils.test.ts` añade un caso para meses sin ingresos usando Vitest, ya existente en el proyecto.
- **financial-dashboard-workflow:** guía interna para integrar vistas financieras respetando los contratos distintos de la API, filtros, cálculos compartidos y estados vacíos/error. Su utilidad práctica está pendiente de validación con una tarea real.

## Verificación y pendientes

- No se ejecutaron tests ni build; no se afirma que pasen.
- No se midió el contraste de los colores ni se completó una revisión manual con teclado o lector de pantalla.
- Pendiente ejecutar los tests y el build, evaluar contraste/accesibilidad en el navegador y probar la skill interna en una tarea real.