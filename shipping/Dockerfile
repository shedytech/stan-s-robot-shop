FROM openjdk:11-jre-slim
WORKDIR /opt/shipping
# Install curl for healthcheck, minimize layer size, and clean up
RUN apt-get update && apt-get install -y --no-install-recommends curl && apt-get clean && rm -rf /var/lib/apt/lists/*
EXPOSE 8080
# Define environment variables with safe defaults
ENV CART_ENDPOINT=cart:8080 \
    DB_HOST=mysql \
    DB_NAME=shipping \
    DB_USER=root \
    DB_PASSWORD=password \
    JAVA_XMS=256m \
    JAVA_XMX=512m
# Copy the built JAR from the build stage
COPY --from=build /opt/shipping/target/*.jar /opt/shipping/shipping.jar
# Create non-root user and set permissions
RUN useradd -m appuser && chown -R appuser:appuser /opt/shipping
USER appuser
# Healthcheck aligned with Kubernetes probes
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 CMD curl --fail http://localhost:8080/health || exit 1
# Use shell form to ensure environment variable substitution
CMD java -Xms${JAVA_XMS:-256m} -Xmx${JAVA_XMX:-512m} -Djava.net.preferIPv4Stack=true -jar /opt/shipping/shipping.jar
