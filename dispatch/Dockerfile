# Build stage
FROM golang:1.23-alpine AS build
WORKDIR /go/src/app
# Copy all source files first
COPY . .
# Initialize Go module if go.mod doesn't exist
RUN if [ ! -f go.mod ]; then go mod init dispatch; fi
RUN go mod tidy && go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o dispatch

# Run stage
FROM alpine:3.18
WORKDIR /app
RUN apk add --no-cache curl
COPY --from=build /go/src/app/dispatch .
RUN adduser -D appuser && chown appuser /app
USER appuser
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD curl --fail --silent http://localhost:8080/health || exit 1
CMD ["./dispatch"]
