---
name: docker
description: Docker containerization platform - Dockerfile, Compose, Build, Networking, Storage, and Security
metadata:
  author: Andreas Sonnleitner
  version: "2026.3.10"
  source: Generated from https://github.com/docker/docs, scripts located at https://github.com/asonnleitner/skills
---

> The skill is based on Docker documentation (docker/docs commit dd3be1e), generated at 2026-03-10.

Docker is a containerization platform for building, shipping, and running applications in isolated containers. This skill covers Dockerfile authoring, Docker Compose configuration, build optimization, networking, storage, security, and CI/CD integration.

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| Dockerfile | Instructions, multi-stage builds, best practices | [core-dockerfile](references/core-dockerfile.md) |
| Compose Services | Service definition, key attributes, syntax | [core-compose-services](references/core-compose-services.md) |
| Compose File Structure | Top-level elements, interpolation, fragments, extensions | [core-compose-file](references/core-compose-file.md) |

## Features

### Build

| Topic | Description | Reference |
|-------|-------------|-----------|
| Build Cache | Cache optimization, invalidation, cache mounts | [feature-build-cache](references/feature-build-cache.md) |
| Build Args & Secrets | ARG, ENV, secrets, SSH mounts, build context | [feature-build-args-secrets](references/feature-build-args-secrets.md) |
| Multi-Platform Builds | Cross-platform images, QEMU, cross-compilation | [feature-build-multiplatform](references/feature-build-multiplatform.md) |

### Compose

| Topic | Description | Reference |
|-------|-------------|-----------|
| Networking | Service discovery, DNS, network drivers, aliases | [feature-compose-networking](references/feature-compose-networking.md) |
| Volumes & Storage | Volumes, bind mounts, tmpfs in Compose | [feature-compose-volumes](references/feature-compose-volumes.md) |
| Environment Variables | env vars, env_file, interpolation, precedence | [feature-compose-environment](references/feature-compose-environment.md) |
| Multiple Files & Profiles | extends, include, merge, profiles, overrides | [feature-compose-multi-file](references/feature-compose-multi-file.md) |
| Deploy & Development | Deploy spec, watch mode, lifecycle hooks, secrets/configs | [feature-compose-deploy-develop](references/feature-compose-deploy-develop.md) |

## Advanced

| Topic | Description | Reference |
|-------|-------------|-----------|
| Docker Bake | HCL-based build orchestration, targets, matrices | [advanced-bake](references/advanced-bake.md) |
| Security & Resources | Rootless mode, capabilities, resource constraints, GPU | [advanced-security-resources](references/advanced-security-resources.md) |
| CI/CD with GitHub Actions | Docker Build in GitHub Actions workflows | [advanced-ci-github-actions](references/advanced-ci-github-actions.md) |
