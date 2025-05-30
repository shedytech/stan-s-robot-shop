# Build stage
FROM alpine AS build
WORKDIR /instana
RUN apk add --update --no-cache wget
ENV ARTI_PATH='artifact-public.instana.io/artifactory/shared/com/instana/nginx_tracing/'
RUN if [ -n "$KEY" ]; then \
    sensor_version=$(wget -O- "https://_:${KEY}@${ARTI_PATH}" 2>&1 | \
      egrep -o "href=\"([0-9]+\.){2}[0-9]+/" | cut -d'"' -f2 | sort -V | tail -n1 | tr -d /); \
    echo "Downloading sensor version ${sensor_version} for Nginx version 1.21.6"; \
    wget -O instana.zip \
      "https://_:${KEY}@${ARTI_PATH}/${sensor_version}/linux-amd64-glibc-nginx-1.21.6.zip" && \
    unzip instana.zip && \
    mv glibc-libinstana_sensor.so libinstana_sensor.so && \
    mv glibc-nginx-1.21.6-ngx_http_ot_module.so ngx_http_opentracing_module.so && \
    rm instana.zip; \
    else echo "KEY not provided. Not adding tracing"; \
    touch dummy.so; \
    fi

# Final stage
FROM nginx:1.21.6
EXPOSE 8080
ENV CATALOGUE_HOST=catalogue \
    USER_HOST=user \
    CART_HOST=cart \
    SHIPPING_HOST=shipping \
    PAYMENT_HOST=payment \
    RATINGS_HOST=ratings \
    INSTANA_SERVICE_NAME=nginx-web

# Copy Instana libraries
COPY --from=build /instana/*.so /tmp/

# Copy configuration and static files
COPY entrypoint.sh /root/
COPY default.conf.template /etc/nginx/conf.d/default.conf.template
COPY static /usr/share/nginx/html

# Set entrypoint
ENTRYPOINT ["/root/entrypoint.sh"]
