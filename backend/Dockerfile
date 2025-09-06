FROM golang:1.25-alpine AS build

RUN apk add --no-cache git

WORKDIR /app

COPY go.* ./
RUN go mod download

COPY . .
# ФЛАГ CGO_ENABLED=0 нужен, чтобы собрать бинарник без зависимостей от C-библиотек (т.к. в финальном образе их не будет)
RUN CGO_ENABLED=0 go build -o /app/binary ./cmd/api

# используем минимальный образ без шелла и пакетных менеджеров для безопасного запуска приложения а также без рут прав
FROM gcr.io/distroless/static-debian12:nonroot

COPY --from=build /app/binary /app/binary

EXPOSE 8081

# тк в приложении есть HTTP endpoint для проверки здоровья, добавим HEALTHCHECK
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD ["/app/binary", "--healthcheck"]

CMD [ "/app/binary" ]