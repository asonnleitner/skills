---
name: core-dockerfile
description: Dockerfile instructions, multi-stage builds, and best practices for writing efficient images
---

# Dockerfile

A Dockerfile is a text file with instructions for building a Docker image. Each instruction creates a layer in the image.

## Key Instructions

```dockerfile
# Base image
FROM node:22-alpine AS base

# Set working directory
WORKDIR /app

# Set build-time variables
ARG NODE_ENV=production

# Set environment variables (persisted in image)
ENV NODE_ENV=$NODE_ENV

# Copy files (prefer COPY over ADD)
COPY package.json package-lock.json ./

# Run commands (each RUN creates a layer)
RUN npm ci --production

# Copy application source
COPY . .

# Document exposed ports (informational only)
EXPOSE 3000

# Set the default user (security best practice)
USER node

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD wget -qO- http://localhost:3000/health || exit 1

# Default command (can be overridden)
CMD ["node", "server.js"]
```

## ENTRYPOINT vs CMD

- **ENTRYPOINT**: The executable that always runs. Use exec form `["executable", "arg"]`.
- **CMD**: Default arguments to ENTRYPOINT, or default command if no ENTRYPOINT. Overridden by `docker run` args.

```dockerfile
# Pattern: ENTRYPOINT for the binary, CMD for default args
ENTRYPOINT ["node"]
CMD ["server.js"]
# docker run myimage          -> node server.js
# docker run myimage app.js   -> node app.js
```

**Shell form vs Exec form:**
- Exec form `["executable", "arg1"]` — runs directly, no shell processing, signals reach process
- Shell form `executable arg1` — runs via `/bin/sh -c`, enables variable expansion but PID 1 is shell

Always use exec form for ENTRYPOINT to ensure proper signal handling.

## Multi-Stage Builds

Use multiple `FROM` statements to create intermediate stages. Only the final stage produces the output image. This dramatically reduces image size.

```dockerfile
# Stage 1: Build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (only this stage ends up in the image)
FROM node:22-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

**Key patterns:**

```dockerfile
# Copy from a specific stage
COPY --from=builder /app/dist ./dist

# Copy from an external image (no need to build it)
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf

# Build only up to a specific stage
# docker build --target builder -t myapp:dev .

# Name stages for clarity
FROM golang:1.23 AS build
FROM gcr.io/distroless/static-debian12 AS production
```

## Best Practices

### Layer Ordering (Cache Efficiency)

Put instructions that change less frequently first:

```dockerfile
FROM node:22-alpine
WORKDIR /app

# 1. Dependencies change less often — cached layer
COPY package.json package-lock.json ./
RUN npm ci

# 2. Source code changes often — invalidates only this and below
COPY . .
RUN npm run build
```

### Minimize Layers

Combine related RUN instructions:

```dockerfile
# Bad: 3 layers
RUN apt-get update
RUN apt-get install -y curl git
RUN rm -rf /var/lib/apt/lists/*

# Good: 1 layer, clean up in same layer
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl git && \
    rm -rf /var/lib/apt/lists/*
```

### Use .dockerignore

Exclude files from the build context to speed up builds and avoid leaking secrets:

```
# .dockerignore
node_modules
.git
.env
*.md
Dockerfile
docker-compose*.yml
.dockerignore
```

### Choose Minimal Base Images

- `alpine` variants are ~5MB vs ~100MB+ for full images
- `distroless` images contain only runtime (no shell, no package manager)
- `scratch` for statically compiled binaries (Go, Rust)

```dockerfile
# For Go
FROM golang:1.23 AS build
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server

FROM scratch
COPY --from=build /app/server /server
ENTRYPOINT ["/server"]
```

### Pin Image Versions

```dockerfile
# Bad: unpredictable
FROM node:latest

# Good: specific version
FROM node:22.14-alpine3.21

# Best: digest for reproducibility
FROM node@sha256:abcdef123456...
```

### Don't Run as Root

```dockerfile
# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

### COPY vs ADD

- Use `COPY` for local files (simple, predictable)
- `ADD` only for tar auto-extraction or remote URLs (rare)

```dockerfile
# Prefer COPY
COPY . .

# ADD auto-extracts tars
ADD archive.tar.gz /app/
```

<!--
Source references:
- https://docs.docker.com/build/concepts/dockerfile/
- https://docs.docker.com/build/building/multi-stage/
- https://docs.docker.com/build/building/best-practices/
- https://docs.docker.com/build/building/base-images/
-->
