---
name: advanced-security-resources
description: Container security, rootless mode, Linux capabilities, resource constraints, and GPU access
---

# Security & Resource Management

## Running as Non-Root

Always run containers as a non-root user:

```dockerfile
# Create user in Dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Or use numeric IDs
USER 1000:1000
```

In Compose:
```yaml
services:
  app:
    image: myapp
    user: "1000:1000"
```

## Linux Capabilities

Docker drops most Linux capabilities by default. Add only what's needed:

```yaml
services:
  app:
    cap_drop:
      - ALL             # drop everything first
    cap_add:
      - NET_BIND_SERVICE  # only add what's needed
```

Common capabilities:
- `NET_BIND_SERVICE` — bind to ports < 1024
- `SYS_PTRACE` — debugging (strace, gdb)
- `CHOWN`, `DAC_OVERRIDE`, `FOWNER` — file ownership operations
- `SETUID`, `SETGID` — change process UID/GID

## Read-Only Filesystem

Prevent container from writing to its filesystem:

```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /run
    volumes:
      - app-data:/app/data    # writable volume for data
```

## Rootless Mode

Run the Docker daemon as a non-root user. Eliminates root privileges for the daemon and containers.

Install: `dockerd-rootless-setuptool.sh install`

Key limitations:
- Port binding below 1024 requires additional configuration
- Some storage drivers are not available
- AppArmor is not supported

## Security Options

```yaml
services:
  app:
    security_opt:
      - no-new-privileges:true    # prevent privilege escalation
      - seccomp=custom.json       # custom seccomp profile
      - apparmor=docker-default   # AppArmor profile
```

## Resource Constraints

### Memory

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M            # hard limit (OOM killed if exceeded)
        reservations:
          memory: 256M            # guaranteed minimum

    # Alternative (non-deploy):
    mem_limit: 512m
    mem_reservation: 256m
    memswap_limit: 1g             # memory + swap total
```

### CPU

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: "1.5"            # 1.5 CPU cores max
        reservations:
          cpus: "0.5"            # guaranteed minimum

    # Alternative (non-deploy):
    cpus: 1.5
    cpu_shares: 512               # relative weight (default 1024)
    cpuset: "0,1"                 # pin to specific CPUs
```

### PIDs Limit

```yaml
services:
  app:
    pids_limit: 100               # prevent fork bombs
```

## GPU Access

```yaml
services:
  ml-training:
    image: pytorch/pytorch
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              capabilities: [gpu]
              count: 2              # number of GPUs

  inference:
    image: mymodel
    gpus: all                       # shorthand: all GPUs
```

Prerequisite: Install NVIDIA Container Toolkit on the host.

## Docker Socket Security

Mounting the Docker socket gives **full host control**:

```yaml
# Dangerous — use with extreme caution
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
```

If needed, consider read-only access or a Docker socket proxy.

## Network Security

```yaml
networks:
  internal:
    internal: true        # no internet access, service-to-service only

services:
  db:
    networks:
      - internal          # database has no internet access
  app:
    networks:
      - internal
      - default           # app has internet + db access
```

## Container Pruning

Clean up unused resources:

```bash
docker system prune                  # dangling images + stopped containers + unused networks
docker system prune -a               # also remove unused images
docker system prune -a --volumes     # also remove unused volumes

# Selective pruning
docker container prune               # stopped containers
docker image prune -a                # unused images
docker volume prune                  # unused volumes
docker network prune                 # unused networks

# With filters
docker image prune -a --filter "until=24h"
docker container prune --filter "until=24h"
```

<!--
Source references:
- https://docs.docker.com/engine/security/
- https://docs.docker.com/engine/security/rootless/
- https://docs.docker.com/engine/containers/resource_constraints/
- https://docs.docker.com/engine/manage-resources/pruning/
- https://docs.docker.com/compose/how-tos/gpu-support/
-->
