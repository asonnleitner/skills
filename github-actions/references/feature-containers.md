---
name: feature-containers
description: GitHub Actions container jobs and service containers
---

# Container Jobs

Run job steps inside a Docker container.

## Job Container

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: node:20-alpine
      credentials:
        username: ${{ github.actor }}
        password: ${{ secrets.GHCR_TOKEN }}
      env:
        NODE_ENV: production
      ports:
        - 8080:80
      volumes:
        - my-data:/data
      options: --cpus 2 --memory 4g

    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

### Container Options

| Key | Description |
|-----|-------------|
| `image` | Docker image to use |
| `credentials` | Registry auth (`username`, `password`) |
| `env` | Container environment variables |
| `ports` | Port mappings (`host:container`) |
| `volumes` | Volume mounts (`source:destination`) |
| `options` | Docker CLI options (except `--network`) |

## Service Containers

Provide backing services (databases, caches) for job steps:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4
      - run: npm test
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
          REDIS_URL: redis://localhost:6379
```

## Networking

### Job on Host (no `container`)

Service containers map ports to the host. Access via `localhost:PORT`:
```yaml
services:
  db:
    image: postgres:16
    ports:
      - 5432:5432  # Required for host access
```

### Job in Container

Services join a Docker bridge network. Access by **service name** as hostname:
```yaml
container:
  image: node:20
services:
  db:
    image: postgres:16
    # No port mapping needed — use hostname "db"
```

```yaml
env:
  DATABASE_URL: postgres://postgres:postgres@db:5432/test
```

## Health Checks

Wait for service readiness before running steps:
```yaml
services:
  postgres:
    image: postgres:16
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

## Private Registry Authentication

```yaml
container:
  image: ghcr.io/owner/custom-image:latest
  credentials:
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

services:
  custom:
    image: registry.example.com/service:v1
    credentials:
      username: ${{ secrets.REGISTRY_USER }}
      password: ${{ secrets.REGISTRY_PASS }}
```

<!--
Source references:
- https://docs.github.com/en/actions/using-containerized-services
- https://docs.github.com/en/actions/using-jobs/running-jobs-in-a-container
-->
