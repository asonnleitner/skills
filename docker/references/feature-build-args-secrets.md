---
name: feature-build-args-secrets
description: Build arguments, environment variables, secrets, SSH mounts, and build context configuration
---

# Build Arguments, Secrets & Context

## ARG vs ENV

- **ARG**: Available only during build. Not persisted in the image.
- **ENV**: Available during build AND at runtime. Persisted in the image.

```dockerfile
# ARG: build-time only
ARG NODE_ENV=production
ARG GIT_COMMIT

# ENV: persisted in image, available at runtime
ENV NODE_ENV=$NODE_ENV
ENV APP_VERSION=1.0.0

# Use ARG to set ENV
ARG BUILD_VERSION=latest
ENV APP_VERSION=$BUILD_VERSION
```

Pass build args at build time:
```bash
docker build --build-arg NODE_ENV=development --build-arg GIT_COMMIT=$(git rev-parse HEAD) .
```

**Scoping:** ARG declared before FROM is only available in FROM. After FROM, redeclare to use in that stage:

```dockerfile
ARG BASE_IMAGE=node:22-alpine
FROM $BASE_IMAGE AS builder

# Must redeclare to use in this stage
ARG NODE_ENV
RUN echo $NODE_ENV
```

## Pre-Defined Build Args

Multi-platform args (auto-set by BuildKit):
- `TARGETPLATFORM` — e.g., `linux/amd64`
- `TARGETOS` — e.g., `linux`
- `TARGETARCH` — e.g., `amd64`
- `TARGETVARIANT` — e.g., `v7`
- `BUILDPLATFORM`, `BUILDOS`, `BUILDARCH`, `BUILDVARIANT` — host platform info

```dockerfile
FROM --platform=$BUILDPLATFORM golang:1.23 AS build
ARG TARGETARCH
RUN GOARCH=$TARGETARCH go build -o /app/server .
```

## Build Secrets

Mount secrets during build without baking them into the image:

```dockerfile
# Mount a secret file
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc \
    npm ci

# Mount and read a secret
RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) && \
    curl -H "Authorization: Bearer $API_KEY" https://api.example.com/data
```

Pass secrets at build time:
```bash
# From file
docker build --secret id=npmrc,src=$HOME/.npmrc .

# From environment variable
docker build --secret id=api_key,env=API_KEY .
```

In Compose:
```yaml
services:
  app:
    build:
      context: .
      secrets:
        - npmrc
secrets:
  npmrc:
    file: ~/.npmrc
```

## SSH Mounts

Forward SSH agent for private repository access during build:

```dockerfile
RUN --mount=type=ssh \
    git clone git@github.com:org/private-repo.git
```

```bash
docker build --ssh default .
# Or specific key:
docker build --ssh default=$SSH_AUTH_SOCK .
```

## Build Context

The build context is the set of files accessible during build.

```bash
# Local directory
docker build .
docker build ./path/to/context

# Git repository
docker build https://github.com/user/repo.git
docker build https://github.com/user/repo.git#branch
docker build https://github.com/user/repo.git#v1.0:subdir

# Tar archive
docker build - < archive.tar.gz
```

### Named Contexts

Reference additional contexts in the build:

```dockerfile
# In Dockerfile
COPY --from=resources /data/config.yaml /app/config.yaml
```

```bash
docker buildx build \
  --build-context resources=./shared/resources \
  --build-context myapp=docker-image://myapp:latest \
  .
```

In Compose:
```yaml
build:
  context: .
  additional_contexts:
    - resources=./shared/resources
    - base=docker-image://mybase:latest
```

### .dockerignore

Exclude files from the build context:

```
# .dockerignore
.git
node_modules
*.md
.env*
Dockerfile
docker-compose*.yml
dist
coverage
.cache
```

Patterns follow `.gitignore` syntax. Use `!` to negate:
```
*.md
!README.md
```

<!--
Source references:
- https://docs.docker.com/build/building/variables/
- https://docs.docker.com/build/building/secrets/
- https://docs.docker.com/build/concepts/context/
-->
