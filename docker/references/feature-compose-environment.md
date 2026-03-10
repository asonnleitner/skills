---
name: feature-compose-environment
description: Environment variables, env_file, interpolation syntax, and precedence rules in Compose
---

# Environment Variables in Compose

## Setting Environment Variables

### In the Compose file

```yaml
services:
  app:
    # Map syntax
    environment:
      NODE_ENV: production
      DB_HOST: db
      DB_PORT: "5432"

    # Array syntax
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - SHELL_VAR             # pass through from host shell
```

### From env_file

```yaml
services:
  app:
    env_file:
      - .env                      # required by default
      - path: ./local.env
        required: false           # silently skip if missing
      - path: ./raw.env
        format: raw               # no interpolation in values
```

**env_file format:**
```bash
# Comments with #
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD="quoted value"
LITERAL='no $expansion'
EMPTY=
# Unset (variable removed):
# UNSET_VAR
```

### At runtime

```bash
docker compose run -e DEBUG=true app
```

## Variable Interpolation in Compose Files

The Compose file itself supports variable substitution from the shell and `.env`:

```yaml
services:
  app:
    image: "myapp:${TAG:-latest}"
    ports:
      - "${HOST_PORT:-8080}:3000"
    environment:
      DB_HOST: ${DB_HOST:?Error: DB_HOST is required}
```

**Substitution syntax:**

| Syntax | Behavior |
|--------|----------|
| `${VAR}` | Value of VAR |
| `${VAR:-default}` | Default if VAR is unset or empty |
| `${VAR-default}` | Default if VAR is unset only |
| `${VAR:?error}` | Error if VAR is unset or empty |
| `${VAR:+replacement}` | Replacement only if VAR is set and non-empty |
| `$$` | Literal `$` (escape) |

**Nesting:** `${VAR:-${FALLBACK:-default}}`

## .env File

Compose auto-loads `.env` from the project directory for Compose file interpolation (not directly injected into containers).

```bash
# .env
TAG=v2.1
HOST_PORT=9090
DB_HOST=localhost
```

Override with CLI:
```bash
docker compose --env-file ./production.env up
```

## Precedence (Highest to Lowest)

1. `docker compose run -e` — CLI override
2. `environment` attribute (with shell/env file interpolation applied)
3. `env_file` attribute
4. Dockerfile `ENV` instruction

When both `environment` and `env_file` set the same variable, `environment` wins:

```yaml
services:
  app:
    env_file: .env          # DB_HOST=from-file
    environment:
      DB_HOST: from-compose # This wins
```

## Key Distinction

- **Interpolation in Compose file** (`${VAR}`): Uses shell env + `.env` file to substitute values in the Compose file itself. Happens before the file is parsed.
- **Container environment** (`environment`/`env_file`): Variables injected into the running container.

```yaml
# .env has: TAG=v2, DB_HOST=localhost
services:
  app:
    image: "myapp:${TAG}"         # Interpolation: myapp:v2
    environment:
      DB_HOST: ${DB_HOST}         # Interpolated then set in container
      APP_SECRET: hardcoded       # Direct value in container
```

## Escaping

Use `$$` to pass a literal `$` to the container:

```yaml
services:
  app:
    command: "echo $$HOME"        # Container sees: echo $HOME
    environment:
      PATTERN: "$$VAR"            # Container sees: $VAR
```

<!--
Source references:
- https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/
- https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/
- https://docs.docker.com/compose/how-tos/environment-variables/envvars-precedence/
- https://docs.docker.com/reference/compose-file/interpolation/
-->
