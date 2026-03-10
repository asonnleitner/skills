---
name: feature-compose-volumes
description: Volumes, bind mounts, and tmpfs in Docker Compose and Engine
---

# Volumes & Storage

## Three Mount Types

| Type | Use Case | Persists | Performance |
|------|----------|----------|-------------|
| **Volume** | Databases, persistent data | Yes, managed by Docker | Best |
| **Bind mount** | Dev config, source code | Host filesystem | Host-dependent |
| **Tmpfs** | Secrets, temp data | No (memory only) | Fastest |

## Named Volumes

Managed by Docker, persist across container restarts and removals.

```yaml
services:
  db:
    image: postgres:17
    volumes:
      - db-data:/var/lib/postgresql/data

  backup:
    image: backup-tool
    volumes:
      - db-data:/data:ro    # read-only access to same volume

volumes:
  db-data:                  # declare at top level
```

### Volume Drivers

```yaml
volumes:
  # NFS volume
  nfs-data:
    driver: local
    driver_opts:
      type: nfs
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/data"

  # CIFS/SMB volume
  cifs-data:
    driver: local
    driver_opts:
      type: cifs
      o: "username=user,password=pass,addr=10.0.0.10"
      device: "//10.0.0.10/share"
```

### External Volumes

Reference a volume created outside of Compose:

```yaml
volumes:
  existing-data:
    external: true          # must already exist
    name: my-existing-vol   # actual volume name
```

## Bind Mounts

Map host directories/files into the container.

```yaml
services:
  app:
    volumes:
      # Short syntax
      - ./src:/app/src              # relative path
      - ./config.yaml:/etc/app/config.yaml:ro  # read-only
      - /var/run/docker.sock:/var/run/docker.sock

      # Long syntax
      - type: bind
        source: ./src
        target: /app/src
        read_only: false
        bind:
          create_host_path: true    # create if missing (default)
```

**When to use bind mounts:**
- Sharing source code during development
- Sharing config files
- Sharing build artifacts from host to container
- Mounting Docker socket

**Caution:** Bind mounts give the container access to the host filesystem. Avoid mounting sensitive directories.

## Tmpfs Mounts

In-memory filesystem. Data is not persisted and not written to disk.

```yaml
services:
  app:
    tmpfs:
      - /run
      - /tmp:size=100m

    # Long syntax in volumes
    volumes:
      - type: tmpfs
        target: /app/temp
        tmpfs:
          size: 52428800    # 50MB
          mode: 1777
```

**When to use tmpfs:**
- Temporary files that shouldn't persist
- Sensitive data that shouldn't be written to disk
- Performance-critical temporary storage

## Volume Subpaths

Mount a subdirectory within a volume:

```yaml
volumes:
  - type: volume
    source: data
    target: /app/config
    volume:
      subpath: config       # mount only data/config/
```

## Common Patterns

### Database with persistent storage

```yaml
services:
  postgres:
    image: postgres:17
    volumes:
      - pg-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_pass
    secrets:
      - db_pass

volumes:
  pg-data:

secrets:
  db_pass:
    file: ./secrets/db_password.txt
```

### Development with live reload

```yaml
services:
  app:
    build: .
    volumes:
      - ./src:/app/src            # live code changes
      - /app/node_modules         # anonymous volume prevents overwrite
    ports:
      - "3000:3000"
```

### Backup and restore

```bash
# Backup a volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar czf /backup/mydata.tar.gz -C /data .

# Restore a volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  tar xzf /backup/mydata.tar.gz -C /data
```

## Volume Management

```bash
docker volume ls                    # list volumes
docker volume inspect mydata        # inspect volume
docker volume rm mydata             # remove volume
docker volume prune                 # remove unused volumes
docker system prune --volumes       # remove all unused data including volumes
```

<!--
Source references:
- https://docs.docker.com/engine/storage/volumes/
- https://docs.docker.com/engine/storage/bind-mounts/
- https://docs.docker.com/engine/storage/tmpfs/
- https://docs.docker.com/reference/compose-file/volumes/
-->
