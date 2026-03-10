---
name: feature-caching
description: GitHub Actions dependency caching with actions/cache and setup-* actions
---

# Dependency Caching

## Using `actions/cache`

```yaml
- uses: actions/cache@v4
  id: cache
  with:
    path: ~/.npm
    key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      npm-${{ runner.os }}-
      npm-
```

### Inputs

| Input | Required | Description |
|-------|----------|-------------|
| `path` | Yes | Path(s) to cache (supports multiline for multiple paths) |
| `key` | Yes | Cache key (max 512 characters) |
| `restore-keys` | No | Fallback keys for partial matches (checked sequentially) |
| `enableCrossOsArchive` | No | Allow cross-OS cache sharing (default: `false`) |

### Output

- `cache-hit` — `true` if exact key match found

```yaml
- if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci
```

## Cache Key Matching

1. Exact match on `key`
2. Prefix match on `key`
3. Sequential prefix match on each `restore-keys` entry
4. If still no match, retry on default branch

On cache miss + successful job completion, a new cache is saved with the provided `key`.

## Setup Actions with Auto-Caching

Most `setup-*` actions support built-in caching:

```yaml
# Node.js (npm, yarn, pnpm)
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'npm'

# Python (pip, pipenv, poetry)
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'

# Go
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true

# Java (gradle, maven)
- uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: 'temurin'
    cache: 'gradle'
```

## Multiple Paths

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
      ~/.m2/repository
    key: build-${{ runner.os }}-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
```

## Cache Scope

- Current branch can access its own caches + default branch caches
- PR caches created on merge ref (`refs/pull/N/merge`) — only accessible by that PR
- Caches cannot be shared across sibling branches

## Limits and Eviction

| Limit | Value |
|-------|-------|
| Per repository | 10 GB default |
| Individual cache | No explicit limit |
| Retention | 7 days without access |
| Upload rate | 200/min per repo |
| Download rate | 1,500/min per repo |

- Least-recently-used caches evicted when limit reached
- Repository cache size can be increased up to 10 TB (may incur charges)

## Key Design Patterns

```yaml
# Lock file hash (exact match)
key: npm-${{ hashFiles('**/package-lock.json') }}

# OS-specific cache
key: cache-${{ runner.os }}-${{ hashFiles('**/lockfile') }}

# Branch-aware cache with fallbacks
key: build-${{ github.ref }}-${{ hashFiles('src/**') }}
restore-keys: |
  build-${{ github.ref }}-
  build-refs/heads/main-
  build-
```

<!--
Source references:
- https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows
-->
