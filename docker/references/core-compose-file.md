---
name: core-compose-file
description: Compose file structure, top-level elements, interpolation, fragments, and extensions
---

# Compose File Structure

The Compose file (preferably named `compose.yaml`) defines a multi-container application.

## Top-Level Elements

```yaml
name: myapp                    # project name

services:                      # container definitions (required)
  web: ...
  db: ...

networks:                      # custom networks
  frontend: ...

volumes:                       # persistent storage
  db-data: ...

configs:                       # configuration data
  app-config: ...

secrets:                       # sensitive data
  db-password: ...
```

## Variable Interpolation

Environment variables from the shell or `.env` file are substituted in the Compose file:

```yaml
services:
  web:
    image: "webapp:${TAG:-latest}"        # default value
    environment:
      DB_HOST: ${DB_HOST:?DB_HOST must be set}  # required, error if unset
      OPTIONAL: ${OPT:+override_value}    # use value only if OPT is set
```

**Syntax:**
- `${VAR}` — substitute value
- `${VAR:-default}` — default if unset or empty
- `${VAR-default}` — default if unset only
- `${VAR:?error}` — error if unset or empty
- `${VAR:+replacement}` — replacement if set and non-empty
- `$$` — escape literal `$` (no interpolation)

Nested interpolation: `${VAR:-${FOO:-default}}`

## .env File

Compose automatically loads `.env` from the project directory for interpolation. Format:

```bash
# Comments start with #
TAG=v2.1
DB_HOST=localhost
DB_PASSWORD="secret with spaces"
LITERAL='no $interpolation here'
# Empty value
EMPTY_VAR=
```

## YAML Fragments (Anchors & Aliases)

Reuse configuration blocks with YAML anchors:

```yaml
x-common-env: &common-env
  environment:
    LOG_LEVEL: info
    TZ: UTC

x-common-deploy: &common-deploy
  deploy:
    resources:
      limits:
        cpus: "0.5"
        memory: 256M

services:
  api:
    <<: *common-env
    <<: *common-deploy
    image: api:latest

  worker:
    <<: *common-env
    <<: *common-deploy
    image: worker:latest
    environment:
      <<: *common-env
      WORKER_CONCURRENCY: "4"    # extend the env block
```

**Note:** YAML `<<` merge only works with mappings (objects), not sequences (arrays).

## Extensions (x- prefix)

Any top-level key starting with `x-` is ignored by Compose but can hold reusable data:

```yaml
x-logging: &default-logging
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

services:
  web:
    <<: *default-logging
    image: nginx

  api:
    <<: *default-logging
    image: api:latest
```

## Networks Top-Level Element

```yaml
networks:
  frontend:
    driver: bridge                    # default driver

  backend:
    driver: bridge
    internal: true                    # no external access
    ipam:
      config:
        - subnet: 172.28.0.0/16

  external-net:
    external: true                    # pre-existing network
    name: my-existing-network         # actual network name
```

## Volumes Top-Level Element

```yaml
volumes:
  db-data:                           # default driver

  nfs-data:
    driver: local
    driver_opts:
      type: nfs
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/data"

  external-vol:
    external: true                   # pre-existing volume
```

## Configs and Secrets Top-Level Elements

```yaml
configs:
  app-config:
    file: ./config.yaml              # from file
  inline-config:
    content: |                       # inline content
      key=value
  env-config:
    environment: CONFIG_VALUE        # from env var

secrets:
  db-password:
    file: ./secrets/db_password.txt  # from file
  api-key:
    environment: API_KEY             # from env var
```

## Project Name

```yaml
name: myapp    # sets project name, used as resource prefix
```

The `version` key is obsolete and ignored. Compose always uses the latest schema.

## Specifying Byte Values and Durations

```yaml
# Byte values: b, k/kb, m/mb, g/gb
mem_limit: 512m
shm_size: 256mb

# Durations: us, ms, s, m, h
stop_grace_period: 1m30s
interval: 500ms
```

<!--
Source references:
- https://docs.docker.com/reference/compose-file/version-and-name/
- https://docs.docker.com/reference/compose-file/interpolation/
- https://docs.docker.com/reference/compose-file/fragments/
- https://docs.docker.com/reference/compose-file/extension/
- https://docs.docker.com/reference/compose-file/networks/
- https://docs.docker.com/reference/compose-file/volumes/
- https://docs.docker.com/reference/compose-file/configs/
- https://docs.docker.com/reference/compose-file/secrets/
-->
