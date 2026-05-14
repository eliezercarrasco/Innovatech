# Innovatech Chile - Sistema de Logística y Ventas

## Descripción del Proyecto

Innovatech Chile es una empresa de logística que requiere un sistema centralizado para gestionar sus operaciones de **despachos** y **ventas**. Este repositorio contiene un monorepo con tres microservicios que conforman la plataforma: un frontend React para la interfaz de usuario y dos APIs REST Spring Boot independientes, cada una con su propia base de datos MySQL.

---

## Arquitectura del Sistema

```
                        ┌──────────────────────────────────────────┐
                        │           innovatech-network              │
                        │                                           │
  Usuario  ──► :80 ──►  │  [frontend]                               │
                        │      │                                    │
                        │      ├──► :8081 ──► [backend-despachos]   │
                        │      │                    │               │
                        │      │              [db-despachos :3306]  │
                        │      │                                    │
                        │      └──► :8082 ──► [backend-ventas]      │
                        │                         │                 │
                        │                   [db-ventas :3306]       │
                        └──────────────────────────────────────────┘
```

- **3 microservicios** comunicados en red interna Docker
- **2 bases de datos MySQL 8.0** con volúmenes nombrados para persistencia
- Frontend desacoplado, consume ambas APIs via HTTP

---

## Tecnologías Utilizadas

| Capa | Tecnología |
|------|-----------|
| Frontend | React 18, Vite 5, TailwindCSS 3, Axios, React Router DOM, SweetAlert2 |
| Backend | Spring Boot 3.4.4, Java 17, Spring Data JPA, SpringDoc OpenAPI, Lombok |
| Base de Datos | MySQL 8.0 |
| Contenedores | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Infraestructura | AWS EC2 |

---

## Estructura del Repositorio

```
Innovatech/
├── docker-compose.yml                          # Orquestación de los 5 servicios
├── README.md
├── .github/
│   └── workflows/
│       ├── deploy-frontend.yml
│       ├── deploy-backend-despachos.yml
│       └── deploy-backend-ventas.yml
├── front_despacho/                             # Frontend React + Vite
│   ├── Dockerfile
│   ├── src/
│   │   ├── componentes/
│   │   └── Routes/
│   ├── package.json
│   └── vite.config.js
├── back-Despachos_SpringBoot/
│   └── Springboot-API-REST-DESPACHO/           # Backend Despachos
│       ├── Dockerfile
│       ├── pom.xml
│       └── src/
└── back-Ventas_SpringBoot/
    └── Springboot-API-REST/                    # Backend Ventas
        ├── Dockerfile
        ├── pom.xml
        └── src/
```

---

## Pre-requisitos

- [Docker](https://docs.docker.com/get-docker/) >= 24.x
- [Docker Compose](https://docs.docker.com/compose/install/) >= 2.x
- [Git](https://git-scm.com/)

---

## Instalación y Ejecución Local

```bash
# Clonar el repositorio
git clone https://github.com/eliezercarrasco/Innovatech.git
cd Innovatech

# Levantar todos los servicios
docker-compose up -d

# Ver logs
docker-compose logs -f

# Detener todos los servicios
docker-compose down
```

Servicios disponibles tras levantar:

| Servicio | URL |
|----------|-----|
| Frontend | http://localhost:80 |
| Backend Despachos | http://localhost:8081 |
| Backend Ventas | http://localhost:8082 |
| MySQL Despachos | localhost:3306 |
| MySQL Ventas | localhost:3307 |

---

## Variables de Entorno

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `MYSQL_ROOT_PASSWORD` | Contraseña root de MySQL | `rootpassword` |
| `MYSQL_DATABASE` | Nombre de la base de datos | `despachos_db` |
| `MYSQL_USER` | Usuario de la base de datos | `despachos_user` |
| `MYSQL_PASSWORD` | Contraseña del usuario | `despachos_pass` |
| `DB_ENDPOINT` | Host de la BD (nombre del servicio Docker) | `db-despachos` |
| `DB_PORT` | Puerto MySQL | `3306` |
| `DB_NAME` | Nombre de la BD en Spring Boot | `despachos_db` |
| `DB_USERNAME` | Usuario de Spring Boot para la BD | `despachos_user` |
| `DB_PASSWORD` | Contraseña de Spring Boot para la BD | `despachos_pass` |

---

## Pipeline CI/CD

El pipeline se activa automáticamente con cada `push` a la rama `deploy`, usando filtros de ruta para detectar qué servicio cambió.

```
Push a rama deploy
        │
        ├── front_despacho/**  ──► deploy-frontend.yml
        │       └── build imagen → push Docker Hub → SSH EC2 → docker run
        │
        ├── back-Despachos_SpringBoot/**  ──► deploy-backend-despachos.yml
        │       └── build imagen → push Docker Hub → SSH EC2 → docker run
        │
        └── back-Ventas_SpringBoot/**  ──► deploy-backend-ventas.yml
                └── build imagen → push Docker Hub → SSH EC2 → docker run
```

### Secrets requeridos en GitHub

| Secret | Descripción |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Usuario de Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acceso Docker Hub |
| `EC2_FRONTEND_HOST` | IP pública de la instancia EC2 frontend |
| `EC2_BACKEND_HOST` | IP pública/privada de la instancia EC2 backend |
| `EC2_SSH_KEY` | Clave privada PEM para SSH |
| `EC2_USERNAME` | Usuario SSH (`ec2-user` o `ubuntu`) |
| `DB_ENDPOINT` | Host de la base de datos |
| `DB_PORT` | Puerto de la base de datos |
| `DB_NAME` | Nombre de la base de datos |
| `DB_USERNAME` | Usuario de la base de datos |
| `DB_PASSWORD` | Contraseña de la base de datos |

---

## Despliegue en AWS EC2

Se utilizan al menos dos instancias EC2:

| Instancia | Rol | Acceso |
|-----------|-----|--------|
| EC2 Frontend | Sirve el frontend Nginx | Pública (Security Group: puerto 80 abierto) |
| EC2 Backend | Ejecuta ambos backends + bases de datos | Privada o pública restringida (puertos 8081, 8082, 3306, 3307) |

**Security Groups recomendados:**
- Frontend: entrada en puerto 80 desde `0.0.0.0/0`, puerto 22 desde IP del equipo
- Backend: entrada en puertos 8081-8082 desde el Security Group del frontend, puerto 22 desde IP del equipo

---

## Endpoints de las APIs

### Backend Despachos (`http://localhost:8081`)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/despachos` | Listar todos los despachos |
| GET | `/api/v1/despachos/{idDespacho}` | Obtener despacho por ID |
| POST | `/api/v1/despachos` | Crear nuevo despacho |
| PUT | `/api/v1/despachos/{idDespacho}` | Actualizar despacho |
| DELETE | `/api/v1/despachos/{idDespacho}` | Eliminar despacho |

Swagger UI: http://localhost:8081/swagger-ui.html

### Backend Ventas (`http://localhost:8082`)

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/ventas` | Listar todas las ventas |
| GET | `/api/v1/ventas/{idVenta}` | Obtener venta por ID |
| POST | `/api/v1/ventas` | Crear nueva venta |
| PUT | `/api/v1/ventas/{idVenta}` | Actualizar venta |
| DELETE | `/api/v1/ventas/{idVenta}` | Eliminar venta |

Swagger UI: http://localhost:8082/swagger-ui.html

---

## Persistencia de Datos

Se utilizan **named volumes** (volúmenes nombrados) en lugar de bind mounts.

### Volúmenes definidos

| Volumen | Servicio | Mount point en contenedor |
|---------|----------|--------------------------|
| `mysql-despachos-data` | `db-despachos` | `/var/lib/mysql` |
| `mysql-ventas-data` | `db-ventas` | `/var/lib/mysql` |

### Por qué named volumes y no bind mounts

| Criterio | Named Volume | Bind Mount |
|----------|-------------|------------|
| Gestión | Docker gestiona permisos y ubicación | Depende del sistema de archivos del host |
| Portabilidad | Funciona igual en Linux, macOS, Windows y EC2 | Requiere que exista el path exacto en el host |
| Rendimiento I/O | Optimizado por Docker | Variable según OS y filesystem |
| Aislamiento | Ciclo de vida independiente del contenedor | Acoplado al directorio del host |
| Despliegue en AWS | Transparente — Docker crea el volumen automáticamente | Requiere crear directorios en EC2 manualmente |

### Demostración de persistencia

```bash
# 1. Levantar el stack
docker compose up -d

# 2. Insertar datos
curl -X POST http://localhost/api/v1/ventas \
  -H "Content-Type: application/json" \
  -d '{"direccionCompra":"Av. Test 123","valorCompra":9990,"fechaCompra":"2026-05-12","despachoGenerado":false}'

# 3. Verificar datos insertados
curl http://localhost/api/v1/ventas
# → [{"idVenta":1,"direccionCompra":"Av. Test 123",...}]

# 4. Bajar contenedores SIN eliminar volúmenes
docker compose down
# Los volúmenes siguen existentes:
docker volume ls   # → evaluacionparcial2_mysql-despachos-data / mysql-ventas-data

# 5. Levantar nuevamente
docker compose up -d

# 6. Verificar que los datos persisten
curl http://localhost/api/v1/ventas
# → [{"idVenta":1,"direccionCompra":"Av. Test 123",...}]  ← dato original intacto
```

### Destrucción controlada

```bash
# Elimina contenedores Y volúmenes (reseteo total de datos)
docker compose down -v

# Después de docker compose up -d los datos estarán vacíos:
curl http://localhost/api/v1/ventas   # → []
```

> **`down` vs `down -v`**: `docker compose down` elimina contenedores y redes pero **preserva** los volúmenes. `docker compose down -v` además elimina los volúmenes nombrados. Usar `-v` solo cuando se quiere un reseteo completo intencional de datos.

### Ubicación física en el host

```bash
docker volume inspect evaluacionparcial2_mysql-despachos-data
# "Mountpoint": "/var/lib/docker/volumes/evaluacionparcial2_mysql-despachos-data/_data"
```

Los datos de MySQL residen en el filesystem de Docker Engine, no en el directorio del proyecto, lo que garantiza que ningún `git clean` o limpieza del workspace los elimine accidentalmente.

---

## Autores

- **Eliezer Carrasco** - [elie.carrasco@duocuc.cl](mailto:elie.carrasco@duocuc.cl)

*Evaluación Parcial N°2 - Herramientas DevOps (ISY1101) - DuocUC*
Trigger pipeline CI/CD