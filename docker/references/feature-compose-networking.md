---
name: feature-compose-networking
description: Docker Compose and Engine networking, service discovery, DNS, and network drivers
---

# Networking in Compose

## Default Network

Compose automatically creates a default network for the project. All services join it and can reach each other by service name.

```yaml
services:
  web:
    image: nginx
    # Can reach "api" at http://api:3000
  api:
    image: node:22-alpine
    # Can reach "db" at db:5432
  db:
    image: postgres:17
```

The default network is named `{project_name}_default`.

## Custom Networks

Isolate groups of services with custom networks:

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: node:22-alpine
    networks:
      - frontend
      - backend

  db:
    image: postgres:17
    networks:
      - backend

networks:
  frontend:
  backend:
```

Here, `web` can reach `api` but not `db`. `api` can reach both.

## Service Discovery & DNS

Services resolve each other by **service name** on shared networks:

```yaml
services:
  api:
    networks:
      backend:
        aliases:
          - api-internal   # additional hostname
          - app-api
```

Other containers on `backend` can reach this service as `api`, `api-internal`, or `app-api`.

## Network Configuration

```yaml
networks:
  frontend:
    driver: bridge

  backend:
    driver: bridge
    internal: true          # no internet access
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"

  custom-ip:
    ipam:
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1

  existing:
    external: true          # use pre-existing network
    name: my-network        # actual network name
```

## Static IP Addresses

```yaml
services:
  app:
    networks:
      custom-ip:
        ipv4_address: 172.28.0.10

networks:
  custom-ip:
    ipam:
      config:
        - subnet: 172.28.0.0/16
```

## Network Drivers

### Bridge (Default)

Isolated network on a single Docker host. Containers communicate via internal IP. Best for most single-host setups.

```bash
docker network create --driver bridge my-network
```

**User-defined bridges vs default bridge:**
- User-defined: automatic DNS resolution by container name
- Default bridge: containers can only communicate by IP
- User-defined: better isolation, containers only reachable if on same network

### Host

Container shares the host's network stack. No network isolation, no port mapping needed.

```yaml
services:
  web:
    image: nginx
    network_mode: host    # binds directly to host ports
```

Use for: performance-critical networking, services needing many ports.

### Overlay

Multi-host networking for Docker Swarm services. Requires open ports: 2377/tcp, 4789/udp, 7946/tcp+udp.

### None

No networking. Container is completely isolated.

```yaml
network_mode: "none"
```

## Port Publishing

```yaml
services:
  web:
    ports:
      # Bind to all interfaces
      - "8080:80"
      # Bind to localhost only (recommended for development)
      - "127.0.0.1:8080:80"
      # Random host port
      - "80"
      # UDP
      - "514:514/udp"
```

## Extra Hosts

Add custom hostname-to-IP mappings:

```yaml
services:
  web:
    extra_hosts:
      - "host.docker.internal=host-gateway"  # access host from container
      - "myhost=192.168.1.100"
```

## Container-to-Host Communication

Access the Docker host from within a container:

```yaml
extra_hosts:
  - "host.docker.internal=host-gateway"
```

Then use `host.docker.internal` as the hostname inside the container.

<!--
Source references:
- https://docs.docker.com/compose/how-tos/networking/
- https://docs.docker.com/engine/network/
- https://docs.docker.com/engine/network/drivers/bridge/
- https://docs.docker.com/engine/network/drivers/host/
- https://docs.docker.com/engine/network/drivers/overlay/
- https://docs.docker.com/engine/network/port-publishing/
-->
