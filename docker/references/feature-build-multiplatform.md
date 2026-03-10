---
name: feature-build-multiplatform
description: Building multi-platform Docker images for different architectures
---

# Multi-Platform Builds

Build images that run on multiple CPU architectures (amd64, arm64, etc.) from a single build command.

## Basic Multi-Platform Build

```bash
# Build for multiple platforms and push to registry
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest \
  --push .
```

The result is a manifest list — one image tag pointing to platform-specific variants. Docker automatically pulls the right one.

## Strategies

### 1. QEMU Emulation (Simplest)

BuildKit uses QEMU to emulate different architectures. No Dockerfile changes needed.

```bash
# Set up QEMU (one-time)
docker run --privileged --rm tonistiigi/binfmt --install all

# Build
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t myapp:latest .
```

Slow for compiled languages but works for any Dockerfile.

### 2. Cross-Compilation (Fastest)

Use the build platform's native speed and cross-compile for the target:

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.23 AS build
ARG TARGETOS TARGETARCH
WORKDIR /app
COPY . .
RUN GOOS=$TARGETOS GOARCH=$TARGETARCH go build -o /app/server .

FROM alpine:3.21
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
```

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .
```

This runs `go build` natively on the build host and cross-compiles — much faster than QEMU.

### 3. Multiple Native Nodes

Use separate builder nodes per architecture:

```bash
docker buildx create --name multibuilder --platform linux/amd64 --node amd64-node
docker buildx create --name multibuilder --append --platform linux/arm64 --node arm64-node
docker buildx use multibuilder
```

## Platform-Specific Behavior

Handle architecture differences in Dockerfile:

```dockerfile
FROM node:22-alpine
ARG TARGETARCH

# Install platform-specific dependency
RUN if [ "$TARGETARCH" = "arm64" ]; then \
      apk add --no-cache some-arm-package; \
    fi

COPY . .
CMD ["node", "server.js"]
```

## Compose Multi-Platform

```yaml
services:
  app:
    build:
      context: .
      platforms:
        - "linux/amd64"
        - "linux/arm64"
    image: myregistry/myapp:latest
```

## Exporting Binaries

Extract build artifacts without creating an image:

```dockerfile
FROM rust:1.83 AS build
WORKDIR /app
COPY . .
RUN cargo build --release

FROM scratch
COPY --from=build /app/target/release/myapp /
```

```bash
# Export to local directory
docker buildx build --output type=local,dest=./out .

# Multi-platform export (creates platform subdirectories)
docker buildx build --platform linux/amd64,linux/arm64 --output type=local,dest=./out .
# Result: out/linux_amd64/myapp, out/linux_arm64/myapp
```

<!--
Source references:
- https://docs.docker.com/build/building/multi-platform/
- https://docs.docker.com/build/building/export/
-->
