# Backend Despachos - API REST Spring Boot (Innovatech Chile)

## Descripción

API REST desarrollada con Spring Boot para la gestión de **despachos** de Innovatech Chile. Expone endpoints CRUD para la entidad Despacho y persiste los datos en una base de datos MySQL 8.0.

---

## Tecnologías

| Tecnología | Versión | Uso |
|-----------|---------|-----|
| Spring Boot | 3.4.4 | Framework principal |
| Java | 17 | Lenguaje de programación |
| MySQL | 8.0 | Base de datos relacional |
| Spring Data JPA | 3.4.3 | Capa de persistencia ORM |
| SpringDoc OpenAPI | 2.7.0 | Documentación Swagger |
| Lombok | — | Reducción de boilerplate |
| Maven | 3.9 | Gestión de dependencias y build |

---

## Endpoints Disponibles

Base URL: `http://localhost:8081`

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/despachos` | Listar todos los despachos |
| GET | `/api/v1/despachos/{idDespacho}` | Obtener despacho por ID |
| POST | `/api/v1/despachos` | Crear nuevo despacho |
| PUT | `/api/v1/despachos/{idDespacho}` | Actualizar despacho existente |
| DELETE | `/api/v1/despachos/{idDespacho}` | Eliminar despacho |

---

## Variables de Entorno Requeridas

| Variable | Descripción | Ejemplo |
|----------|-------------|---------|
| `DB_ENDPOINT` | Host de la base de datos MySQL | `db-despachos` / `localhost` |
| `DB_PORT` | Puerto de MySQL | `3306` |
| `DB_NAME` | Nombre de la base de datos | `despachos_db` |
| `DB_USERNAME` | Usuario de la base de datos | `despachos_user` |
| `DB_PASSWORD` | Contraseña del usuario | `despachos_pass` |

Estas variables son leídas en `application.properties` mediante la sintaxis `${DB_ENDPOINT}`.

---

## Ejecución Local

```bash
# Requiere Java 17 y Maven instalados, y MySQL corriendo localmente

# Establecer variables de entorno
export DB_ENDPOINT=localhost
export DB_PORT=3306
export DB_NAME=despachos_db
export DB_USERNAME=root
export DB_PASSWORD=password

# Ejecutar la aplicación
mvn spring-boot:run

# O bien compilar y ejecutar el JAR
mvn clean package -DskipTests
java -jar target/Springboot-API-REST-DESPACHO-0.0.1-SNAPSHOT.jar
```

---

## Ejecución con Docker

```bash
# Construir la imagen
docker build -t innovatech-backend-despachos .

# Ejecutar el contenedor
docker run -d \
  --name backend-despachos \
  -p 8081:8081 \
  -e DB_ENDPOINT=<host_mysql> \
  -e DB_PORT=3306 \
  -e DB_NAME=despachos_db \
  -e DB_USERNAME=despachos_user \
  -e DB_PASSWORD=despachos_pass \
  innovatech-backend-despachos
```

El Dockerfile usa una construcción multi-stage:
1. **Stage builder** (`maven:3.9-eclipse-temurin-17`): descarga dependencias y compila el JAR omitiendo tests
2. **Stage runtime** (`eclipse-temurin:17-jre-alpine`): imagen ligera JRE que ejecuta el JAR con usuario no root

---

## Swagger UI

Documentación interactiva de la API disponible en:

```
http://localhost:8081/swagger-ui.html
```

---

## Puerto

La aplicación escucha en el puerto **8081** (configurado en `application.properties`).

---

## CI/CD

El pipeline se activa automáticamente al hacer push a la rama `deploy` con cambios en `back-Despachos_SpringBoot/**`.

| Etapa | Descripción |
|-------|-------------|
| `build-and-push` | Compila la imagen Docker y la publica en Docker Hub con tags `:latest` y `:<git-sha>` |
| `deploy-to-ec2` | Descarga la imagen y la ejecuta en la instancia EC2 (requiere secretos EC2 configurados) |

Imagen publicada: `DOCKERHUB_USERNAME/innovatech-backend-despachos`
