# Etapa 1: Build
FROM eclipse-temurin:21-jdk-alpine AS build

WORKDIR /app

# Copia todo el proyecto (asegúrate de construir con el contexto en la raíz del repo)
COPY . .

# (Opcional) Si tu build depende de submódulos y el contexto incluye .git, inicialízalos
# Si no existe .git, este paso no falla gracias al '|| true'
RUN apk add --no-cache git && \
    [ -d .git ] && git submodule update --init --recursive || true

# Gradle Wrapper ejecutable
RUN chmod +x ./gradlew

# Compila omitiendo tests y la validación de estructura para evitar falsos positivos en CI
# Ajusta los flags según tu pipeline
RUN ./gradlew clean build -x test -x validateStructure

# Normaliza el nombre del artefacto a app.jar para simplificar el COPY en la etapa runtime
# Busca primero un bootJar, si no existe toma el primer .jar disponible
RUN set -eux; \
    JAR="$(ls applications/app-service/build/libs/*-SNAPSHOT.jar 2>/dev/null || true)"; \
    [ -z "$JAR" ] && JAR="$(ls applications/app-service/build/libs/*.jar | head -n 1)"; \
    cp "$JAR" /app/app.jar

# Etapa 2: Runtime
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Copia el artefacto generado
COPY --from=build /app/app.jar ./app.jar

# Puerto típico en Spring Boot (ajústalo si usas otro)
EXPOSE 8080

# Opciones JVM recomendadas en contenedores
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=70 -Djava.security.egd=file:/dev/./urandom"

# Arranque
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
