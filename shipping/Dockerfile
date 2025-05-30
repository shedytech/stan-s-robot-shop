# Build stage
FROM maven:3.8-openjdk-11-slim AS build
WORKDIR /opt/shipping
COPY pom.xml .
COPY src ./src
RUN mvn -B package --no-transfer-progress

# Run stage
FROM openjdk:11-jre-slim
WORKDIR /opt/shipping
RUN apt-get update && apt-get install -y --no-install-recommends curl && rm -rf /var/lib/apt/lists/*
EXPOSE 8080
ENV CART_ENDPOINT=cart:8080 \
    DB_HOST=mysql \
    DB_NAME=shipping \
    DB_USER=root \
    DB_PASSWORD=password \
    JAVA_XMS=256m \
    JAVA_XMX=768m
COPY --from=build /opt/shipping/target/*.jar shipping.jar
RUN useradd -m appuser && chown -R appuser /opt/shipping
USER appuser
HEALTHCHECK --interval=30s --timeout=10s CMD curl --fail http://localhost:8080/health || exit 1
CMD ["java", "-Xms${JAVA_XMS}", "-Xmx${JAVA_XMX}", "-jar", "shipping.jar"]

