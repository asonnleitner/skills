---
name: feature-compose-deploy-develop
description: Deploy specification, resource limits, watch mode, lifecycle hooks, secrets, and configs
---

# Deploy, Development & Lifecycle

## Deploy Specification

Controls runtime resources, replicas, and restart behavior:

```yaml
services:
  app:
    image: myapp
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
      restart_policy:
        condition: on-failure    # none | on-failure | any
        delay: 5s
        max_attempts: 3
        window: 120s
      update_config:
        parallelism: 2
        delay: 10s
        order: start-first       # start-first | stop-first
```

### GPU Access

```yaml
services:
  ml:
    image: pytorch/pytorch
    deploy:
      resources:
        reservations:
          devices:
            - capabilities: [gpu]
              count: 1            # or "all"
              driver: nvidia

# Shorthand syntax
services:
  ml:
    image: pytorch/pytorch
    gpus: all
```

## Watch Mode (Development)

Automatically sync files or rebuild on changes (Compose 2.22.0+):

```yaml
services:
  frontend:
    build: ./frontend
    develop:
      watch:
        # Sync source files to container
        - path: ./frontend/src
          action: sync
          target: /app/src
          ignore:
            - node_modules/
            - "**/*.test.js"

        # Rebuild on dependency changes
        - path: ./frontend/package.json
          action: rebuild

        # Sync + restart on config change
        - path: ./frontend/config
          action: sync+restart
          target: /app/config

        # Sync + execute a command
        - path: ./frontend/locales
          action: sync+exec
          target: /app/locales
          exec:
            command: npm run compile-i18n
```

**Actions:**
| Action | Behavior |
|--------|----------|
| `sync` | Copy changed files to container |
| `rebuild` | Rebuild image and recreate container |
| `restart` | Restart container (no rebuild) |
| `sync+restart` | Sync files then restart |
| `sync+exec` | Sync files then run a command |

```bash
docker compose watch      # start watching
docker compose up --watch  # start + watch
```

## Lifecycle Hooks

Run commands on container start/stop:

```yaml
services:
  app:
    image: myapp

    # Run after container starts
    post_start:
      - command: /app/init-data.sh
        user: root
        privileged: true
      - command: /app/warm-cache.sh

    # Run before container stops
    pre_stop:
      - command: /app/graceful-shutdown.sh
        environment:
          TIMEOUT: "30"
```

## Secrets in Services

Mount sensitive data as files (not environment variables):

```yaml
services:
  db:
    image: postgres:17
    secrets:
      - db_password                    # mounted at /run/secrets/db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

  app:
    image: myapp
    secrets:
      - source: api_key
        target: /app/secrets/api.key   # custom mount path
        mode: 0440

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    environment: API_KEY               # from host env var
```

## Configs in Services

Mount non-sensitive configuration data:

```yaml
services:
  web:
    image: nginx
    configs:
      - source: nginx_conf
        target: /etc/nginx/nginx.conf

  app:
    image: myapp
    configs:
      - app_config                     # mounted at /app_config

configs:
  nginx_conf:
    file: ./nginx/nginx.conf
  app_config:
    content: |
      debug=false
      port=3000
```

## Startup Order with Health Checks

```yaml
services:
  db:
    image: postgres:17
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  migrations:
    image: myapp
    command: ["npm", "run", "migrate"]
    depends_on:
      db:
        condition: service_healthy

  app:
    image: myapp
    depends_on:
      db:
        condition: service_healthy
        restart: true                  # restart app when db restarts
      migrations:
        condition: service_completed_successfully
```

## Production Tips

```yaml
# compose.prod.yaml
services:
  app:
    restart: always
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 1G
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
    # Remove dev volumes
    volumes: !override []
```

```bash
# Deploy with rebuild
docker compose -f compose.yaml -f compose.prod.yaml up -d --build

# Update single service without downtime
docker compose -f compose.yaml -f compose.prod.yaml up -d --no-deps --build app
```

<!--
Source references:
- https://docs.docker.com/reference/compose-file/deploy/
- https://docs.docker.com/reference/compose-file/develop/
- https://docs.docker.com/compose/how-tos/file-watch/
- https://docs.docker.com/compose/how-tos/lifecycle/
- https://docs.docker.com/compose/how-tos/use-secrets/
- https://docs.docker.com/compose/how-tos/production/
- https://docs.docker.com/compose/how-tos/startup-order/
-->
