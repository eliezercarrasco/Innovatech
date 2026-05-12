FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml ./
RUN mvn dependency:go-offline -B

COPY src ./src
RUN mvn clean package -DskipTests -B

FROM eclipse-temurin:17-jre-alpine AS runtime

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

COPY --from=builder /app/target/Springboot-API-REST-0.0.1-SNAPSHOT.jar app.jar

RUN chown appuser:appgroup app.jar

USER appuser

ENV DB_ENDPOINT="" \
    DB_PORT="3306" \
    DB_NAME="" \
    DB_USERNAME="" \
    DB_PASSWORD=""

EXPOSE 8081

ENTRYPOINT ["java", "-jar", "app.jar"]
