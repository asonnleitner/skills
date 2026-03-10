---
name: advanced-ci-github-actions
description: Docker Build integration with GitHub Actions for CI/CD pipelines
---

# Docker Build in GitHub Actions

## Basic Build and Push

```yaml
name: Build and Push
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: user/app:latest
```

## Caching in CI

### GitHub Actions Cache

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: user/app:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### Registry Cache

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: user/app:latest
    cache-from: type=registry,ref=user/app:cache
    cache-to: type=registry,ref=user/app:cache,mode=max
```

## Multi-Platform Build

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: user/app:latest
```

## Automatic Tag Management

Use `docker/metadata-action` for semver-based tagging:

```yaml
- name: Docker meta
  id: meta
  uses: docker/metadata-action@v5
  with:
    images: user/app
    tags: |
      type=ref,event=branch
      type=ref,event=pr
      type=semver,pattern={{version}}
      type=semver,pattern={{major}}.{{minor}}
      type=sha

- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: ${{ github.event_name != 'pull_request' }}
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

Tag patterns:
- `type=semver,pattern={{version}}` — `v1.2.3` → `1.2.3`
- `type=semver,pattern={{major}}.{{minor}}` — `v1.2.3` → `1.2`
- `type=ref,event=branch` — branch name as tag
- `type=sha` — git commit SHA
- `type=raw,value=latest,enable={{is_default_branch}}` — `latest` on main

## Build Secrets

```yaml
- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: user/app:latest
    secrets: |
      "npmrc=${{ secrets.NPM_TOKEN }}"
      "github_token=${{ secrets.GITHUB_TOKEN }}"
```

In Dockerfile:
```dockerfile
RUN --mount=type=secret,id=npmrc,target=/root/.npmrc npm ci
```

## Test Before Push

```yaml
- name: Build for testing
  uses: docker/build-push-action@v6
  with:
    context: .
    load: true                        # load into local Docker
    tags: myapp:test

- name: Run tests
  run: docker run --rm myapp:test npm test

- name: Build and push
  if: success()
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: user/app:latest
```

## Share Image Between Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - name: Build and export
        uses: docker/build-push-action@v6
        with:
          context: .
          tags: myapp:latest
          outputs: type=docker,dest=/tmp/myapp.tar
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: myapp
          path: /tmp/myapp.tar

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: myapp
          path: /tmp
      - name: Load image
        run: docker load --input /tmp/myapp.tar
      - name: Test
        run: docker run --rm myapp:latest npm test
```

## Push to Multiple Registries

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

- name: Login to GHCR
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.repository_owner }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Build and push
  uses: docker/build-push-action@v6
  with:
    context: .
    push: true
    tags: |
      user/app:latest
      ghcr.io/user/app:latest
```

<!--
Source references:
- https://docs.docker.com/build/ci/github-actions/configure-builder/
- https://docs.docker.com/build/ci/github-actions/cache/
- https://docs.docker.com/build/ci/github-actions/multi-platform/
- https://docs.docker.com/build/ci/github-actions/secrets/
- https://docs.docker.com/build/ci/github-actions/manage-tags-labels/
- https://docs.docker.com/build/ci/github-actions/test-before-push/
- https://docs.docker.com/build/ci/github-actions/share-image-jobs/
-->
