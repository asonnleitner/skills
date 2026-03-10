---
name: feature-compose-multi-file
description: Multiple Compose files, extends, include, merge rules, profiles, and override patterns
---

# Multiple Compose Files & Profiles

## Override Files

By default, Compose loads `compose.yaml` and merges `compose.override.yaml` if present:

```yaml
# compose.yaml (base)
services:
  app:
    image: myapp:latest
    ports:
      - "8080:80"

# compose.override.yaml (dev overrides, auto-loaded)
services:
  app:
    build: .
    volumes:
      - ./src:/app/src
    environment:
      DEBUG: "true"
```

Explicit multiple files:
```bash
docker compose -f compose.yaml -f compose.prod.yaml up
```

## Merge Rules

### Mappings (environment, labels, build.args)
Keys from the override replace matching keys; new keys are added:
```yaml
# Base:     environment: { A: 1, B: 2 }
# Override: environment: { B: 9, C: 3 }
# Result:   environment: { A: 1, B: 9, C: 3 }
```

### Sequences (ports, expose, cap_add)
Values are concatenated:
```yaml
# Base:     ports: ["8080:80"]
# Override: ports: ["9090:90"]
# Result:   ports: ["8080:80", "9090:90"]
```

**Exception:** `command`, `entrypoint`, and `healthcheck.test` are **replaced**, not appended.

### Unique resources (ports, volumes, secrets, configs)
Merged by unique key (target path for volumes, target+protocol for ports).

### Reset and Override Tags

```yaml
# Reset a value completely
services:
  app:
    ports: !reset []              # remove all ports
    environment:
      FOO: !reset null            # remove specific key

# Override (replace, don't merge)
services:
  app:
    ports: !override
      - "8443:443"                # replaces all ports
```

## extends

Reuse service configuration from another service:

```yaml
# common.yaml
services:
  base:
    image: node:22-alpine
    environment:
      LOG_LEVEL: info

# compose.yaml
services:
  api:
    extends:
      file: common.yaml
      service: base
    ports:
      - "3000:3000"
    environment:
      PORT: "3000"          # merged with base environment

  worker:
    extends:
      file: common.yaml
      service: base
    command: ["node", "worker.js"]
```

Within the same file:
```yaml
services:
  base:
    image: node:22-alpine
    environment:
      LOG_LEVEL: info

  api:
    extends:
      service: base
    ports:
      - "3000:3000"
```

## include

Load external Compose files as separate application models:

```yaml
# compose.yaml
include:
  - ./infra/compose.yaml            # short syntax
  - path: ./monitoring/compose.yaml  # long syntax
    env_file: ./monitoring/.env
  - path:                            # merge multiple files
      - ./db/compose.yaml
      - ./db/compose.override.yaml

services:
  app:
    depends_on:
      - db                # defined in included file
```

- Included files are loaded as independent models
- Relative paths resolve from the included file's directory
- Resource name conflicts produce warnings

## Profiles

Conditionally activate services:

```yaml
services:
  app:
    image: myapp               # always started (no profile)

  debug:
    image: debug-tools
    profiles:
      - debug                  # only started with --profile debug

  test:
    image: test-runner
    profiles:
      - test

  seed:
    image: seed-data
    profiles:
      - dev
      - test                   # multiple profiles
```

Activate profiles:
```bash
docker compose --profile debug up
docker compose --profile debug --profile test up

# Via environment variable
COMPOSE_PROFILES=debug,test docker compose up

# Explicitly targeting a profiled service enables its profile
docker compose up debug
```

**Note:** `depends_on` does NOT auto-enable profiles. If a non-profiled service depends on a profiled service, the profiled service must be explicitly activated.

## Common Patterns

### Base + Environment Overrides

```
compose.yaml              # base config
compose.override.yaml     # dev (auto-loaded)
compose.prod.yaml         # production
compose.test.yaml         # testing
```

```bash
# Development (auto-loads override)
docker compose up

# Production
docker compose -f compose.yaml -f compose.prod.yaml up

# Testing
docker compose -f compose.yaml -f compose.test.yaml up
```

### Modular Infrastructure

```yaml
# compose.yaml
include:
  - infra/database/compose.yaml
  - infra/cache/compose.yaml
  - infra/monitoring/compose.yaml

services:
  app:
    build: .
    depends_on:
      - postgres
      - redis
```

<!--
Source references:
- https://docs.docker.com/compose/how-tos/multiple-compose-files/merge/
- https://docs.docker.com/compose/how-tos/multiple-compose-files/extends/
- https://docs.docker.com/compose/how-tos/multiple-compose-files/include/
- https://docs.docker.com/compose/how-tos/profiles/
- https://docs.docker.com/reference/compose-file/merge/
- https://docs.docker.com/reference/compose-file/profiles/
- https://docs.docker.com/reference/compose-file/include/
-->
