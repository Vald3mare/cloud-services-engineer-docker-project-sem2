FROM golang:1.25-alpine AS build

RUN apk add --no-cache git

WORKDIR /app

COPY go.* ./
RUN go mod download

COPY . .
# ФЛАГ CGO_ENABLED=0 нужен, чтобы собрать бинарник без зависимостей от C-библиотек (т.к. в финальном образе их не будет)
RUN CGO_ENABLED=0 go build -o /app/binary -trimpath -buildvcs=false -ldflags="-s -w" ./cmd/api

# используем минимальный образ без шелла и пакетных менеджеров для безопасного запуска приложения а также без рут прав
FROM alpine:3.20

RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app

RUN apk add --no-cache ca-certificates wget

COPY --from=build /app/binary /app/binary
RUN chown -R appuser:appgroup /app

USER appuser
EXPOSE 8081

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD wget -q -O- http://127.0.0.1:8081/health >/dev/null 2>&1 || exit 1

ENTRYPOINT ["/app/binary"]