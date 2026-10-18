# Uso del sistema de variables CSS del proyecto

**Cuándo se aplica:** Al crear o modificar cualquier componente visual, añadir nuevos estilos CSS, o cambiar la paleta de colores del dashboard.

## Regla

El proyecto define sus colores mediante variables CSS personalizadas en el espacio de color `oklch()`, no con las clases de color estándar de Tailwind (como `bg-blue-500` o `text-red-600`). Esto permite mantener coherencia visual y soportar modo oscuro.

### Variables disponibles

Las variables están definidas en `frontend/src/index.css`. Las principales son:

| Variable | Propósito |
|---|---|
| `--background` / `--foreground` | Fondo y texto principal |
| `--card` / `--card-foreground` | Fondo y texto de tarjetas |
| `--primary` / `--primary-foreground` | Color primario (acento azul) |
| `--secondary` / `--secondary-foreground` | Color secundario |
| `--muted` / `--muted-foreground` | Texto secundario/atenuado |
| `--destructive` / `--destructive-foreground` | Errores y acciones destructivas |
| `--border` | Color de bordes |
| `--chart-income` / `--chart-outcome` / `--chart-profit` | Colores de gráficos |
| `--income-badge` / `--outcome-badge` / `--profit-badge` | Fondos de insignias KPI |

### Cómo usarlas

En las clases Tailwind, las variables se aplican con la notación `bg-[var(--variable)]` o `text-[var(--variable)]`, o mediante las clases semánticas ya definidas:

```typescript
// Correcto: usa variables del proyecto
className={cn('bg-[var(--primary)] text-[var(--primary-foreground)]', className)}
className="bg-primary text-foreground text-muted-foreground"
```

```typescript
// Incorrecto: valores fijos que ignoran el sistema de temas
className="bg-blue-500 text-white"
```

### Modo oscuro

El proyecto tiene dos variantes de color:
- **Modo claro:** valores por defecto en `:root` (línea 5 de `index.css`)
- **Modo oscuro:** valores en el bloque `.dark` (línea 28 de `index.css`)

El modo oscuro se activa añadiendo la clase `dark` a un elemento contenedor (como se hace en `frontend/src/App.tsx` línea 38: `<main className="dark …">`).

### Cómo añadir un nuevo color

Si necesitas un color que no existe entre las variables:

1. Añádelo como variable CSS en `frontend/src/index.css` usando el espacio de color `oklch()`.
2. Defínelo tanto en `:root` (modo claro) como en `.dark` (modo oscuro).
3. Sigue la nomenclatura existente: usa nombres descriptivos con el prefijo `--` y sufijos como `-foreground` cuando aplique (ej. `--mi-color` / `--mi-color-foreground`).

## Qué hacer

1. Usa siempre las variables CSS del proyecto para colores. No uses valores fijos de Tailwind.
2. Si necesitas un color nuevo, añádelo como variable en `index.css` con un valor `oklch()` en ambos bloques (`:root` y `.dark`).
3. Después de cambios visuales, verifica que el dashboard se vea correctamente tanto en modo claro como en modo oscuro.

## Archivos de referencia

- `frontend/src/index.css` — definición de todas las variables CSS (líneas 5-35)
- `frontend/src/components/dashboard/kpi-card.tsx` — ejemplo de uso de variables (líneas 27-41)
- `frontend/src/App.tsx` — activación del modo oscuro (línea 38)