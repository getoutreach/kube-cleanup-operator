# syntax=docker/dockerfile:1.0.0-experimental
FROM gcr.io/outreach-docker/alpine/golang:latest as builder

# Default to using go modules and go proxy
ENV GO111MODULE on
ENV GOPRIVATE github.com/getoutreach/*
ENV GOPROXY https://proxy.golang.org

ARG VERSION
WORKDIR /src

# Ensure that we're using the latest git, openssh, and ca-certs
RUN apk add --no-cache --update \
  openssh-client \
  git \
  ca-certificates

# Add go.mod and go.sum first to maximize caching
COPY ./go.mod ./go.sum ./
RUN --mount=type=ssh set -e -x; \
  mkdir -p ~/.ssh; \
  printf "Host github.com\n  User git" > ~/.ssh/config; \
  ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts; \
  git config --global url."git@github.com:".insteadOf "https://github.com/"; \
  go mod download

COPY go.mod .
COPY go.sum .

RUN go mod download

COPY . /build
WORKDIR /build
RUN go test ./...
RUN make install_deps
RUN make build

FROM gcr.io/outreach-docker/alpine:3.11
MAINTAINER Sergey Nuzhdin <ipaq.lw@gmail.com>

RUN addgroup -S kube-operator && adduser -S -g kube-operator kube-operator

USER kube-operator

COPY --from=build /build/bin/kube-cleanup-operator .

ENTRYPOINT ["./kube-cleanup-operator"]
