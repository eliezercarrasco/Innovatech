# Frontend - Sistema de Despachos (Innovatech Chile)

## Descripción

Aplicación web SPA desarrollada en React para la gestión integral de **despachos** y **ventas** de Innovatech Chile. Provee una interfaz de usuario moderna con operaciones CRUD completas para ambas entidades, consumiendo las APIs REST de los backends Spring Boot.

---

## Tecnologías

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| React | 18.2 | Librería UI principal |
| Vite | 5.2 | Bundler y servidor de desarrollo |
| TailwindCSS | 3.4 | Estilos utilitarios |
| Axios | 1.6 | Cliente HTTP para APIs |
| React Router DOM | 6.24 | Navegación SPA |
| SweetAlert2 | 11 | Alertas y confirmaciones |
| React Hook Form | 7.52 | Gestión de formularios |
| React Icons | 5.1 | Iconografía |

---

## Instalación Local

```bash
# Instalar dependencias
npm install

# Iniciar servidor de desarrollo (http://localhost:5173)
npm run dev
```

---

## Build de Producción

```bash
# Generar build optimizado
npm run build
```

El comando genera la carpeta `dist/` con los artefactos estáticos listos para ser servidos por Nginx u otro servidor web.

```bash
# Previsualizar el build localmente
npm run preview
```

---

## Ejecución con Docker

```bash
# Construir la imagen
docker build -t innovatech-frontend .

# Ejecutar el contenedor
docker run -d -p 80:80 --name frontend innovatech-frontend

# Acceder en http://localhost:80
```

El Dockerfile usa una construcción multi-stage:
1. **Stage builder** (`node:18-alpine`): instala dependencias y genera el build con Vite
2. **Stage runtime** (`nginx:alpine`): sirve los archivos estáticos del directorio `dist/`

---

## Variables de Entorno

Las URLs de los backends se configuran en el código fuente de Axios. Para cambiar los endpoints en producción, actualizar las URLs base en los servicios de Axios:

| Variable | Descripción | Valor por defecto (local) |
|----------|-------------|--------------------------|
| URL Backend Despachos | Host del backend despachos | `http://localhost:8081` |
| URL Backend Ventas | Host del backend ventas | `http://localhost:8082` |

---

## Estructura de Carpetas

```
front_despacho/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── componentes/
│   │   └── CrudAdmin.jsx       # Componente principal CRUD
│   ├── Routes/
│   │   └── AppRoutes.jsx       # Definición de rutas
│   ├── index.css               # Estilos globales + directivas Tailwind
│   └── main.jsx                # Punto de entrada React
├── Dockerfile
├── index.html                  # HTML raíz de Vite
├── package.json
├── vite.config.js
├── tailwind.config.js
└── postcss.config.js
```

---

## Integración con APIs Backend

El frontend consume los siguientes endpoints mediante Axios:

### Backend Despachos (`/api/v1/despachos`)

| Operación | Método | Endpoint |
|-----------|--------|----------|
| Listar despachos | GET | `/api/v1/despachos` |
| Crear despacho | POST | `/api/v1/despachos` |
| Actualizar despacho | PUT | `/api/v1/despachos/{idDespacho}` |
| Eliminar despacho | DELETE | `/api/v1/despachos/{idDespacho}` |

### Backend Ventas (`/api/v1/ventas`)

| Operación | Método | Endpoint |
|-----------|--------|----------|
| Listar ventas | GET | `/api/v1/ventas` |
| Crear venta | POST | `/api/v1/ventas` |
| Actualizar venta | PUT | `/api/v1/ventas/{idVenta}` |
| Eliminar venta | DELETE | `/api/v1/ventas/{idVenta}` |

---

## CI/CD

El pipeline se activa automáticamente al hacer push a la rama `deploy` con cambios en `front_despacho/**`.

| Etapa | Descripción |
|-------|-------------|
| `build-and-push` | Ejecuta `npm run build` dentro de Docker y publica la imagen en Docker Hub con tags `:latest` y `:<git-sha>` |
| `deploy-to-ec2` | Descarga la imagen y la ejecuta en la instancia EC2 con nginx en puerto 8080 (requiere secretos EC2 configurados) |

Imagen publicada: `DOCKERHUB_USERNAME/innovatech-frontend`
