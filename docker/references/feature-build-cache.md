---
name: feature-build-cache
description: Build cache optimization, invalidation strategies, and cache mounts for faster builds
---

# Build Cache

Docker build cache stores layers from previous builds. When a layer's inputs haven't changed, it's reused instead of rebuilt.

## How Cache Works

Each Dockerfile instruction creates a layer. BuildKit checks if the instruction and its inputs match a cached layer:
- **Cache hit**: Layer is reused (fast)
- **Cache miss**: Layer and all subsequent layers are rebuilt

Once a cache miss occurs, all following layers are also rebuilt — order matters.

## Layer Ordering for Cache Efficiency

Put instructions that change **less frequently** first:

```dockerfile
FROM node:22-alpine
WORKDIR /app

# 1. Package files change infrequently
COPY package.json package-lock.json ./
RUN npm ci

# 2. Source code changes often (only rebuilds from here)
COPY . .
RUN npm run build
```

## Cache Mounts

Mount a persistent cache directory for package managers. The cache persists across builds without being stored in the image.

```dockerfile
# Node.js
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Python
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

# Go
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go build -o /app/server .

# Apt
RUN --mount=type=cache,target=/var/cache/apt \
    --mount=type=cache,target=/var/lib/apt/lists \
    apt-get update && apt-get install -y curl
```

## Bind Mounts (Avoid Copying)

Mount files read-only during build without copying them into the image:

```dockerfile
# Mount source for build, don't COPY it
RUN --mount=type=bind,source=go.sum,target=go.sum \
    --mount=type=bind,source=go.mod,target=go.mod \
    go mod download

# Then copy only what's needed
COPY . .
RUN go build -o /app/server .
```

## Cache Invalidation Rules

- **COPY/ADD**: Cache invalidated if file contents (checksum) change
- **RUN**: Cache invalidated if the command string changes or any previous layer was invalidated
- **ARG**: Changing a build arg value invalidates layers that reference it
- **Secrets**: Secrets do NOT invalidate cache (by design)

## Cache Busting

Force a fresh install when needed:

```dockerfile
# Use a build arg to bust cache selectively
ARG CACHEBUST=1
RUN echo "cache bust: $CACHEBUST" && npm ci
# Change CACHEBUST value to force rebuild:
# docker build --build-arg CACHEBUST=$(date +%s) .
```

## External Cache

Export/import cache for CI environments:

```dockerfile
# Use registry as cache backend
docker buildx build \
  --cache-from type=registry,ref=myrepo/app:cache \
  --cache-to type=registry,ref=myrepo/app:cache,mode=max \
  .

# GitHub Actions cache backend
docker buildx build \
  --cache-from type=gha \
  --cache-to type=gha,mode=max \
  .

# Local cache directory
docker buildx build \
  --cache-from type=local,src=/tmp/.buildx-cache \
  --cache-to type=local,dest=/tmp/.buildx-cache-new,mode=max \
  .
```

`mode=max` caches all layers (including intermediate stages). `mode=min` (default) caches only the final stage.

## Keep Build Context Small

Use `.dockerignore` to exclude unnecessary files:

```
node_modules
.git
.env
*.md
dist
build
.DS_Store
```

A smaller context means faster transfer to the build daemon and fewer cache invalidations.

<!--
Source references:
- https://docs.docker.com/build/cache/
- https://docs.docker.com/build/cache/optimize/
- https://docs.docker.com/build/cache/invalidation/
- https://docs.docker.com/build/concepts/context/
-->
