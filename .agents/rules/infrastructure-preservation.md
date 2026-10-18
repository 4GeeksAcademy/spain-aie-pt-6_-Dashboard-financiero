# Preservación de la infraestructura (Docker y Vite proxy)

**Cuándo se aplica:** Al modificar `frontend/vite.config.ts`, `docker-compose.yml`, cualquiera de los `Dockerfile`, o al añadir nuevos servicios de infraestructura.

## Regla

La comunicación entre frontend y backend depende de una configuración específica que debe mantenerse coherente.

### Proxy de Vite

El archivo `frontend/vite.config.ts` (líneas 10-15) define un proxy que redirige las peticiones a `/api` hacia el backend:

```typescript
proxy: {
    "/api": {
        target: "http://backend:8000",
        changeOrigin: true,
    },
},
```

El target `http://backend:8000` funciona porque **Docker Compose** crea una red interna donde el nombre de servicio `backend` se resuelve por DNS. Fuera de Docker este target no funciona. Para entornos sin Docker se usa la variable `VITE_API_BASE_URL` (ver abajo).

### Variable de entorno para orígenes alternativos

En `frontend/src/App.tsx` (línea 13):

```typescript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL ?? "";
```

Si se define `VITE_API_BASE_URL`, se antepone a las rutas de la API. Si no, se usa cadena vacía y el proxy de Vite se encarga. Al añadir nuevas rutas de API, asegúrate de que usan el prefijo `/api` para que el proxy las cubra.

### Puertos expuestos

| Servicio | Puerto | Propósito | Configurado en |
|---|---|---|---|
| Frontend | 5173 | Servidor de desarrollo Vite | `frontend/Dockerfile` línea 8, `docker-compose.yml` línea 6 |
| Backend API | 8000 | FastAPI + Uvicorn | `backend/Dockerfile` línea 11, `docker-compose.yml` línea 10 |
| Debugger | 5678 | debugpy | `backend/Dockerfile` línea 11, `docker-compose.yml` línea 11 |

### Volúmenes para desarrollo en caliente

- `docker-compose.yml` líneas 7-8: el frontend monta `./frontend:/app` y usa un volumen anónimo `/app/node_modules` para evitar sobrescribir los módulos del contenedor con los del host.
- `docker-compose.yml` líneas 14-15: el backend monta `./backend:/app`.
- El backend usa el flag `--reload` (backend `Dockerfile` línea 11) para reinicio automático ante cambios.

## Qué hacer

1. **No cambiar el target del proxy** de `backend:8000` a `localhost:8000` sin verificar el impacto. Si necesitas soportar un entorno sin Docker, usa la variable `VITE_API_BASE_URL`.

2. **No eliminar ni cambiar los puertos** sin actualizar tanto el `Dockerfile` como `docker-compose.yml` y cualquier documentación que los referencie.

3. **No quitar los volúmenes bind mount** ni el volumen anónimo de `node_modules`. Son necesarios para el desarrollo en caliente.

4. **Si añades un nuevo servicio** (base de datos, cache, etc.), considera que la aplicación actual funciona completamente en memoria con datos mock. El nuevo servicio debería ser opt-in o tener la mock-data como alternativa por defecto para desarrollo y tests.

5. **Verifica el funcionamiento** después de cualquier cambio de infraestructura:
   - Ejecuta `docker compose up --build` y confirma que ambos servicios arrancan sin errores.
   - Comprueba que `http://localhost:5173` carga el dashboard.
   - Comprueba que `http://localhost:8000/health` devuelve `{"status":"ok"}`.
   - Si modificaste rutas de API, verifica que el proxy sigue redirigiendo correctamente.

## Archivos de referencia

- `frontend/vite.config.ts` — configuración del proxy
- `frontend/src/App.tsx` — variable `VITE_API_BASE_URL`
- `docker-compose.yml` — definición de servicios, puertos y volúmenes
- `backend/Dockerfile` — comando de inicio, puertos y flag `--reload`
- `frontend/Dockerfile` — comando de inicio y puerto