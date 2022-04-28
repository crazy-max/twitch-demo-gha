# syntax=docker/dockerfile:1

FROM --platform=$BUILDPLATFORM crazymax/goreleaser-xx:latest AS goreleaser-xx
FROM --platform=$BUILDPLATFORM golang:1.18-alpine AS base
COPY --from=goreleaser-xx / /
RUN apk add --no-cache file git
WORKDIR /src

FROM base AS build
ARG TARGETPLATFORM
ARG GIT_REF
RUN --mount=type=bind,source=.,target=/src,rw \
  --mount=type=cache,target=/root/.cache/go-build \
  --mount=type=cache,target=/go/pkg/mod \
  goreleaser-xx --debug \
    --name="twitch-gha-demo" \
    --dist="/out" \
    --hooks="go mod tidy" \
    --hooks="go mod download" \
    --main="." \
    --ldflags="-s -w -X 'main.version={{.Version}}' -X main.commit={{.Commit}} -X main.date={{.Date}}" \
    --files="LICENSE" \
    --files="README.md"

FROM scratch AS artifact
COPY --from=build /out/*.tar.gz /
COPY --from=build /out/*.zip /

FROM alpine
RUN apk --update --no-cache add ca-certificates libressl
COPY --from=build /usr/local/bin/twitch-gha-demo /usr/local/bin/twitch-gha-demo
EXPOSE 8080
ENTRYPOINT [ "twitch-gha-demo" ]
