# syntax=docker/dockerfile:1.7

FROM golang:1.26-alpine AS build

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

# Copia os diretórios necessários para compilar o monolito
COPY internal ./internal
COPY monolito ./monolito

ARG TARGETOS=linux
ARG TARGETARCH=amd64
ARG APP_VERSION=dev

# Compila o binário apontando para o diretório do monolito
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH \
    go build -trimpath -ldflags="-s -w -X github.com/elton-bt/gotodolist/internal/config.version=${APP_VERSION}" -o /out/gotodolist-monolito ./monolito

FROM alpine:3.22.4 AS runtime

# Upgrade de pacotes forçado para mitigar vulnerabilidades críticas do Alpine antes de adicionar o usuário
RUN apk update && apk upgrade --no-cache && \
    addgroup -S app && adduser -S app -G app && \
    apk add --no-cache ca-certificates wget

WORKDIR /app

COPY --from=build /out/gotodolist-monolito /app/gotodolist-monolito

EXPOSE 8080

USER app

ENTRYPOINT ["/app/gotodolist-monolito"]