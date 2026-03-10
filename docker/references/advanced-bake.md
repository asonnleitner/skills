---
name: advanced-bake
description: Docker Bake for HCL-based build orchestration with targets, variables, and matrices
---

# Docker Bake

Bake is a build orchestration tool built into Buildx. It uses HCL (or YAML/JSON) files to define complex build configurations declaratively.

## Basic Usage

```hcl
// docker-bake.hcl
target "app" {
  context    = "."
  dockerfile = "Dockerfile"
  tags       = ["myapp:latest"]
}
```

```bash
docker buildx bake app           # build specific target
docker buildx bake               # build "default" target
docker buildx bake app tests     # build multiple targets
```

## Targets

A target maps to a `docker build` invocation:

```hcl
target "webapp" {
  context    = "./webapp"
  dockerfile = "Dockerfile.prod"
  tags       = ["registry.example.com/webapp:latest", "registry.example.com/webapp:v1.0"]
  args = {
    NODE_ENV = "production"
  }
  platforms = ["linux/amd64", "linux/arm64"]
  target    = "production"    # multi-stage target
  cache-from = ["type=registry,ref=registry.example.com/webapp:cache"]
  cache-to   = ["type=registry,ref=registry.example.com/webapp:cache,mode=max"]
}
```

### Default and Group Targets

```hcl
# Built when no target is specified
target "default" {
  context = "."
  tags    = ["myapp:latest"]
}

# Build multiple targets at once
group "all" {
  targets = ["webapp", "api", "worker"]
}

group "test" {
  targets = ["unit-tests", "integration-tests"]
}
```

## Variables

```hcl
variable "TAG" {
  default = "latest"
}

variable "REGISTRY" {
  default = "docker.io/myuser"
}

target "app" {
  tags = ["${REGISTRY}/app:${TAG}"]
}
```

Override at build time:
```bash
docker buildx bake --set "*.tags=myapp:dev"
docker buildx bake app --set app.args.NODE_ENV=development
TAG=v2.0 docker buildx bake
```

## Inheritance

Targets can inherit from other targets:

```hcl
target "_common" {
  dockerfile = "Dockerfile"
  args = {
    GO_VERSION = "1.23"
  }
}

target "app" {
  inherits = ["_common"]
  context  = "./app"
  tags     = ["myapp:latest"]
}

target "api" {
  inherits = ["_common"]
  context  = "./api"
  tags     = ["myapi:latest"]
}
```

Convention: prefix base targets with `_` to indicate they're not built directly.

## Matrices

Build multiple variants from a single target:

```hcl
target "app" {
  name = "app-${item.os}-${item.arch}"
  matrix = {
    item = [
      { os = "linux",   arch = "amd64" },
      { os = "linux",   arch = "arm64" },
      { os = "windows", arch = "amd64" },
    ]
  }
  platforms = ["${item.os}/${item.arch}"]
  tags     = ["myapp:latest-${item.os}-${item.arch}"]
}
```

## HCL Expressions and Functions

```hcl
variable "TAG" {
  default = "latest"
}

# Conditional
target "app" {
  tags = [
    "myapp:${TAG}",
    TAG != "latest" ? "myapp:latest" : "",
  ]
}

# Functions
target "app" {
  tags = [
    "myapp:${replace(TAG, "/", "-")}",
  ]
  labels = {
    "build.date" = timestamp()
  }
}
```

Available functions: `upper`, `lower`, `replace`, `substr`, `trimspace`, `regex_replace`, `join`, `split`, `timestamp`, etc.

## Named Contexts

Use external sources as build contexts:

```hcl
target "app" {
  contexts = {
    shared   = "./shared-libs"
    baseimg  = "docker-image://mybase:latest"
    upstream = "https://github.com/user/repo.git#main"
  }
}
```

In Dockerfile:
```dockerfile
COPY --from=shared /libs /app/libs
COPY --from=baseimg /etc/config /app/config
```

## Using with Docker Compose

Bake can read `docker-compose.yml` as a build definition:

```bash
docker buildx bake -f docker-compose.yml
```

Combine with HCL overrides:
```bash
docker buildx bake -f docker-compose.yml -f docker-bake.hcl
```

<!--
Source references:
- https://docs.docker.com/build/bake/introduction/
- https://docs.docker.com/build/bake/targets/
- https://docs.docker.com/build/bake/variables/
- https://docs.docker.com/build/bake/matrices/
- https://docs.docker.com/build/bake/inheritance/
- https://docs.docker.com/build/bake/expressions/
- https://docs.docker.com/build/bake/contexts/
-->
