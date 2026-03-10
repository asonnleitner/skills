---
name: core-compose-services
description: Docker Compose service definition with all key attributes and syntax patterns
---

# Compose Services

Services are defined under the `services` top-level key. Each service defines a container configuration.

## Basic Service

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    environment:
      - NGINX_HOST=example.com

  api:
    build:
      context: ./api
      target: production
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:
```

## image

```yaml
image: redis:7
image: redis@sha256:0ed5d5928d...
image: my_private.registry:5000/redis
```

If omitted, `build` must be present.

## build

```yaml
# Short syntax
build: ./dir

# Full syntax
build:
  context: ./backend
  dockerfile: Dockerfile.prod
  target: production
  args:
    GIT_COMMIT: cdc3b19
  platforms:
    - "linux/amd64"
    - "linux/arm64"

# Inline Dockerfile
build:
  context: .
  dockerfile_inline: |
    FROM node:22-alpine
    COPY . .
    CMD ["node", "index.js"]
```

## ports

```yaml
# Short syntax: [HOST:]CONTAINER[/PROTOCOL]
ports:
  - "8080:80"           # host 8080 -> container 80
  - "3000"              # random host port -> container 3000
  - "127.0.0.1:8001:8001"  # bind to localhost only
  - "6060:6060/udp"     # UDP protocol

# Long syntax
ports:
  - target: 80
    published: "8080"
    host_ip: 127.0.0.1
    protocol: tcp
```

**Always quote port mappings** to avoid YAML parsing issues with the colon.

## volumes

```yaml
# Short syntax: [SOURCE:]TARGET[:MODE]
volumes:
  - db-data:/var/lib/postgresql/data     # named volume
  - ./config:/etc/app/config:ro          # bind mount, read-only
  - /var/run/docker.sock:/var/run/docker.sock  # host path

# Long syntax
volumes:
  - type: volume
    source: db-data
    target: /data
    volume:
      subpath: sub
  - type: bind
    source: ./config
    target: /etc/app/config
    read_only: true
```

## environment and env_file

```yaml
# Map syntax
environment:
  NODE_ENV: production
  DB_HOST: db

# Array syntax
environment:
  - NODE_ENV=production
  - DB_HOST=db
  - SHELL_VAR    # pass from host shell

# External file
env_file:
  - .env
  - path: ./override.env
    required: false
```

`environment` values override `env_file` values.

## depends_on

```yaml
# Short: start order only
depends_on:
  - db
  - redis

# Long: with health conditions
depends_on:
  db:
    condition: service_healthy
    restart: true         # restart this service when db restarts
  redis:
    condition: service_started
  migrations:
    condition: service_completed_successfully
```

Conditions: `service_started`, `service_healthy`, `service_completed_successfully`.

## healthcheck

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s

# Disable healthcheck from image
healthcheck:
  disable: true
```

Test formats:
- `["CMD", "executable", "arg"]` — run directly
- `["CMD-SHELL", "command"]` — run via shell
- `command string` — equivalent to CMD-SHELL

## restart

```yaml
restart: "no"             # default, never restart
restart: always           # always restart
restart: on-failure       # restart on non-zero exit
restart: on-failure:3     # max 3 retries
restart: unless-stopped   # restart unless manually stopped
```

## command and entrypoint

```yaml
# Override CMD
command: bundle exec thin -p 3000
command: ["node", "server.js"]    # exec form

# Override ENTRYPOINT
entrypoint: /code/entrypoint.sh
entrypoint: ["php", "-d", "memory_limit=-1"]
```

**Note:** `command` doesn't run in a shell by default. For shell features (variable expansion), wrap it:
```yaml
command: /bin/sh -c 'echo "hello $$HOSTNAME"'
```

## networks

```yaml
services:
  web:
    networks:
      - frontend
      - backend

  api:
    networks:
      backend:
        aliases:
          - api-internal
        ipv4_address: 172.16.238.10

networks:
  frontend:
  backend:
```

Services without `networks` connect to the implicit `default` network.

## Other Key Attributes

```yaml
services:
  app:
    container_name: my-app          # custom name (prevents scaling)
    hostname: app-host              # container hostname
    working_dir: /app               # override WORKDIR
    user: "1000:1000"               # run as specific user
    init: true                      # use init process (PID 1)
    read_only: true                 # read-only filesystem
    stdin_open: true                # equivalent to -i
    tty: true                       # equivalent to -t
    platform: linux/amd64           # target platform
    pull_policy: missing            # always|never|missing|build
    stop_grace_period: 30s          # time before SIGKILL
    stop_signal: SIGTERM            # signal to stop container
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    labels:
      com.example.description: "My app"
    extra_hosts:
      - "host.docker.internal=host-gateway"
    dns:
      - 8.8.8.8
    cap_add:
      - SYS_PTRACE
    cap_drop:
      - NET_ADMIN
    sysctls:
      net.core.somaxconn: 1024
    ulimits:
      nofile:
        soft: 20000
        hard: 40000
    tmpfs:
      - /run
      - /tmp
```

<!--
Source references:
- https://docs.docker.com/reference/compose-file/services/
- https://docs.docker.com/reference/compose-file/build/
-->
