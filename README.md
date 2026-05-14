# Innovatech — Sistema de Logística y Ventas

Plataforma de microservicios para gestión de despachos y ventas desplegada en AWS EC2 con pipeline CI/CD automatizado.

---

## Integrantes

| Nombre | Correo |
|--------|--------|
| Eliezer Carrasco | elie.carrasco@duocuc.cl |
| Maria Jose Velazques | mariajose.velazques@duocuc.cl |

*Evaluación Parcial N°2 — Herramientas DevOps (ISY1101) — DuocUC*

---

## Descripción del Proyecto

Innovatech Chile es una empresa de logística que requiere un sistema centralizado para gestionar sus operaciones de **despachos** y **ventas**. La plataforma está compuesta por tres microservicios containerizados, una red Docker interna y un pipeline CI/CD que automatiza el despliegue en AWS EC2.

### Componentes principales

| Componente | Descripción |
|---|---|
| **Frontend** | Aplicación React/Vite servida por Nginx. Interfaz de usuario para gestión de ventas y despachos. Hace proxy inverso hacia ambos backends. |
| **Backend Despachos** | API REST Spring Boot. Gestiona el ciclo de vida de los despachos logísticos. Expone Swagger UI. |
| **Backend Ventas** | API REST Spring Boot. Gestiona registros de ventas. Expone Swagger UI. |
| **MySQL Despachos** | Base de datos exclusiva para el servicio de despachos. Datos persistidos en named volume. |
| **MySQL Ventas** | Base de datos exclusiva para el servicio de ventas. Datos persistidos en named volume. |

---

## Tecnologías Utilizadas

| Capa | Tecnología | Versión |
|---|---|---|
| Frontend | React + Vite + TailwindCSS | 18 / 5 / 3 |
| Backend | Spring Boot + Java | 3.4.4 / 17 |
| Base de datos | MySQL | 8.0 |
| Contenedores | Docker + Docker Compose | 27.x / 2.x |
| Servidor web | Nginx (Alpine) | latest |
| CI/CD | GitHub Actions | — |
| Registry | Docker Hub | — |
| Infraestructura | AWS EC2 (Ubuntu 24.04) | — |
| Documentación API | SpringDoc OpenAPI (Swagger) | 2.7 |

---

## Arquitectura del Sistema

```
  Internet
      │
      ▼ :80
┌─────────────────────────────────────────────────┐
│                innovatech-network                │
│                  (bridge)                        │
│                                                  │
│  ┌─────────────┐                                 │
│  │  frontend   │  Nginx + React SPA              │
│  │  :8080      │  envsubst → proxy_pass dinámico │
│  └──────┬──────┘                                 │
│         │                                        │
│    ┌────┴─────────────────────┐                  │
│    │                          │                  │
│    ▼ /api/v1/despachos        ▼ /api/v1/ventas   │
│  ┌────────────────┐  ┌───────────────────┐       │
│  │backend-despacho│  │  backend-ventas   │       │
│  │ Spring Boot    │  │  Spring Boot      │       │
│  │ :8081          │  │  :8081 → host8082 │       │
│  └───────┬────────┘  └────────┬──────────┘       │
│          │                    │                  │
│  ┌───────▼────────┐  ┌────────▼──────────┐       │
│  │  db-despachos  │  │    db-ventas      │       │
│  │  MySQL 8.0     │  │    MySQL 8.0      │       │
│  │  :3306         │  │    :3306          │       │
│  └───────┬────────┘  └────────┬──────────┘       │
│          │                    │                  │
└──────────┼────────────────────┼──────────────────┘
           ▼                    ▼
   mysql-despachos-data   mysql-ventas-data
      (named volume)        (named volume)
```

### Características de diseño

- **Red interna Bridge**: todos los contenedores se comunican por nombre de servicio Docker, sin IPs hardcodeadas.
- **Nginx como API Gateway**: el frontend hace proxy inverso hacia los backends usando `envsubst` para configurar hostnames en tiempo de arranque del contenedor.
- **Bases de datos independientes**: cada microservicio tiene su propia instancia MySQL, evitando acoplamiento de datos.
- **Usuarios no-root**: todos los Dockerfiles usan usuarios sin privilegios de root.
- **Multi-stage builds**: imágenes finales livianas (Alpine) separadas del entorno de compilación.

---

## Estructura del Repositorio

```
Innovatech/
├── docker-compose.yml              # Stack local (build desde fuente)
├── docker-compose.prod.yml         # Stack producción (imágenes Docker Hub)
├── .env.example                    # Variables de entorno documentadas
├── .github/
│   └── workflows/
│       ├── deploy-frontend.yml
│       ├── deploy-backend-despachos.yml
│       └── deploy-backend-ventas.yml
├── front_despacho/
│   ├── Dockerfile
│   ├── nginx.conf                  # Template con envsubst
│   ├── src/
│   └── package.json
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/
│       ├── Dockerfile
│       ├── pom.xml
│       └── src/
└── back-Ventas_SpringBoot/
    └── Springboot-API-REST/
        ├── Dockerfile
        ├── pom.xml
        └── src/
```

---

## Pipeline CI/CD

Cada push a `main` o `deploy` dispara automáticamente el pipeline correspondiente según los archivos modificados.

```
git push origin main
        │
        ├── front_despacho/**           → CI/CD Frontend
        │       ├── 1. docker buildx build ./front_despacho
        │       ├── 2. docker push eliezercarrasco/innovatech-frontend:latest
        │       └── 3. SSH EC2 → docker compose pull + up --no-deps frontend
        │
        ├── back-Despachos_SpringBoot/** → CI/CD Backend Despachos
        │       ├── 1. docker buildx build ./back-Despachos_SpringBoot/...
        │       ├── 2. docker push eliezercarrasco/innovatech-backend-despachos:latest
        │       └── 3. SSH EC2 → docker compose pull + up --no-deps backend-despachos
        │
        └── back-Ventas_SpringBoot/**   → CI/CD Backend Ventas
                ├── 1. docker buildx build ./back-Ventas_SpringBoot/...
                ├── 2. docker push eliezercarrasco/innovatech-backend-ventas:latest
                └── 3. SSH EC2 → docker compose pull + up --no-deps backend-ventas
```

### Secrets requeridos en GitHub

Configurar en: **Settings → Secrets and variables → Actions**

| Secret | Descripción | Valor |
|---|---|---|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub | `eliezercarrasco` |
| `DOCKERHUB_TOKEN` | Access Token Docker Hub | Generado en hub.docker.com |
| `EC2_FRONTEND_HOST` | IP pública EC2 (frontend) | `34.229.249.107` |
| `EC2_BACKEND_HOST` | IP pública EC2 (backends) | `34.229.249.107` |
| `EC2_USERNAME` | Usuario SSH de EC2 | `ubuntu` |
| `EC2_SSH_KEY` | Contenido completo del archivo `.pem` | `-----BEGIN RSA PRIVATE KEY-----...` |

### Imágenes en Docker Hub

| Imagen | Tag |
|---|---|
| `eliezercarrasco/innovatech-frontend` | `latest` + SHA del commit |
| `eliezercarrasco/innovatech-backend-despachos` | `latest` + SHA del commit |
| `eliezercarrasco/innovatech-backend-ventas` | `latest` + SHA del commit |

---

## Docker — Contenedores y Healthchecks

| Contenedor | Imagen | Puerto host | Healthcheck |
|---|---|---|---|
| `frontend` | `nginx:alpine` (multi-stage) | `80` | `wget http://127.0.0.1:8080/` |
| `backend-despachos` | `eclipse-temurin:17-jre-alpine` | `8081` | `wget /api/v1/despachos` |
| `backend-ventas` | `eclipse-temurin:17-jre-alpine` | `8082` | `wget /api/v1/ventas` |
| `db-despachos` | `mysql:8.0` | — (solo interno) | `mysqladmin ping` |
| `db-ventas` | `mysql:8.0` | — (solo interno) | `mysqladmin ping` |

La cadena de arranque garantizada por `depends_on: service_healthy`:

```
db-despachos (healthy) ──► backend-despachos (healthy) ──┐
                                                          ├──► frontend
db-ventas    (healthy) ──► backend-ventas    (healthy) ──┘
```

### Persistencia con Named Volumes

| Volumen | Contenedor | Datos |
|---|---|---|
| `mysql-despachos-data` | `db-despachos` | `/var/lib/mysql` |
| `mysql-ventas-data` | `db-ventas` | `/var/lib/mysql` |

Los datos sobreviven a `docker compose down`. Solo se eliminan con `docker compose down -v`.

---

## Ejecución Local

### Pre-requisitos

- Docker >= 24.x
- Docker Compose >= 2.x
- Git

### Comandos

```bash
# 1. Clonar el repositorio
git clone https://github.com/eliezercarrasco/Innovatech.git
cd Innovatech

# 2. Levantar todos los servicios (build desde código fuente)
docker compose up -d

# 3. Verificar que todos los contenedores estén healthy
docker compose ps

# 4. Ver logs en tiempo real
docker compose logs -f

# 5. Detener sin borrar datos
docker compose down

# 6. Detener y borrar datos (reset completo)
docker compose down -v
```

### URLs locales

| Servicio | URL |
|---|---|
| Frontend | http://localhost |
| Swagger Despachos | http://localhost:8081/swagger-ui.html |
| Swagger Ventas | http://localhost:8082/swagger-ui.html |

---

## Despliegue en AWS EC2

### Setup inicial (una sola vez)

```bash
# Conectar a la instancia
ssh -i innovatech-key.pem ubuntu@34.229.249.107

# Instalar dependencias
sudo apt-get update && sudo apt-get install -y docker.io docker-compose-v2 git
sudo usermod -aG docker ubuntu && newgrp docker

# Clonar el repositorio
git clone https://github.com/eliezercarrasco/Innovatech.git /home/ubuntu/innovatech
cd /home/ubuntu/innovatech

# Levantar el stack de producción (imágenes desde Docker Hub)
docker compose -f docker-compose.prod.yml up -d

# Verificar estado
docker compose -f docker-compose.prod.yml ps
```

### Deploy automático (CI/CD)

Después del setup inicial, cada push al repositorio actualiza automáticamente la EC2. También se puede disparar manualmente desde:

**GitHub → Actions → [workflow] → Run workflow**

---

## Endpoints de las APIs

### Backend Despachos — `http://34.229.249.107:8081`

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/v1/despachos` | Listar todos los despachos |
| `GET` | `/api/v1/despachos/{id}` | Obtener despacho por ID |
| `POST` | `/api/v1/despachos` | Crear nuevo despacho |
| `PUT` | `/api/v1/despachos/{id}` | Actualizar despacho |
| `DELETE` | `/api/v1/despachos/{id}` | Eliminar despacho |

### Backend Ventas — `http://34.229.249.107:8082`

| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/api/v1/ventas` | Listar todas las ventas |
| `GET` | `/api/v1/ventas/{id}` | Obtener venta por ID |
| `POST` | `/api/v1/ventas` | Crear nueva venta |
| `PUT` | `/api/v1/ventas/{id}` | Actualizar venta |
| `DELETE` | `/api/v1/ventas/{id}` | Eliminar venta |

---

## URLs Públicas (AWS EC2)

| Servicio | URL |
|---|---|
| Frontend | http://34.229.249.107 |
| Swagger Despachos | http://34.229.249.107:8081/swagger-ui.html |
| Swagger Ventas | http://34.229.249.107:8082/swagger-ui.html |

---

## Estado Actual Validado

| Componente | Estado |
|---|---|
| 5 contenedores Docker | healthy |
| Frontend React/Nginx | HTTP 200 OK |
| API Despachos | HTTP 200 OK |
| API Ventas | HTTP 200 OK |
| Swagger Despachos | Operativo |
| Swagger Ventas | Operativo |
| CI/CD GitHub Actions | Operativo |
| Persistencia MySQL | Validada (down → up con datos intactos) |
| Docker networking bridge | Operativo |
| Named volumes | Operativos |
| envsubst Nginx | Operativo |
| AWS EC2 | Desplegado y accesible públicamente |
| Docker Hub | 3 imágenes publicadas |
