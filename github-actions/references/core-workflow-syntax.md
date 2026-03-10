---
name: core-workflow-syntax
description: Complete GitHub Actions workflow YAML syntax reference
---

# Workflow Syntax

Workflows are YAML files in `.github/workflows/`. Each workflow must have at least `on` and `jobs`.

## Top-Level Keys

```yaml
name: CI                          # Display name in GitHub UI
run-name: Deploy by @${{ github.actor }}  # Custom name per run (supports expressions)

on: push                          # Event trigger(s)

permissions:                      # GITHUB_TOKEN scopes
  contents: read
  pull-requests: write

env:                              # Workflow-level env vars
  NODE_ENV: production

defaults:
  run:
    shell: bash
    working-directory: ./src

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  build:
    # ...
```

## Jobs

```yaml
jobs:
  build:
    name: Build App                # Display name
    runs-on: ubuntu-latest         # Runner label(s)
    needs: [setup, lint]           # Job dependencies
    if: github.ref == 'refs/heads/main'  # Conditional execution
    timeout-minutes: 30            # Max runtime (default/max: 360)
    continue-on-error: false       # Don't fail workflow if job fails
    permissions:
      contents: read
    environment:
      name: production
      url: https://example.com
    outputs:
      version: ${{ steps.ver.outputs.version }}
    env:
      CI: true
    defaults:
      run:
        shell: bash
    steps:
      # ...
```

### `runs-on` Options

```yaml
runs-on: ubuntu-latest              # GitHub-hosted (ubuntu-24.04, ubuntu-22.04)
runs-on: windows-latest             # Windows
runs-on: macos-latest               # macOS
runs-on: [self-hosted, linux, x64]  # Self-hosted with labels
```

## Steps

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4       # Action reference

  - name: Setup Node
    uses: actions/setup-node@v4
    with:                           # Action inputs
      node-version: 20

  - name: Install
    run: npm ci                     # Shell command
    shell: bash                     # Override shell
    working-directory: ./app
    env:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
    if: success()                   # Conditional
    continue-on-error: false
    timeout-minutes: 10
    id: install                     # ID for referencing in contexts
```

### Action Reference Formats

```yaml
uses: actions/checkout@v4                    # Major version tag
uses: actions/checkout@v4.2.0               # Specific version
uses: actions/checkout@8f4b7f84864484a7...  # Commit SHA (most secure)
uses: actions/checkout@main                  # Branch
uses: ./.github/actions/my-action            # Local action
uses: docker://alpine:3.8                    # Docker Hub image
uses: docker://ghcr.io/owner/image          # Container registry
```

### Shell Options

| Shell | Behavior |
|-------|----------|
| `bash` | `set -eo pipefail` (fail-fast) |
| `sh` | `set -e` (fail on error) |
| `pwsh` | `$ErrorActionPreference = 'stop'` |
| `python` | Runs as Python script |
| `cmd` | Windows Command Prompt |

Override fail-fast: `shell: bash {0}` (removes `set -e`)

## Permissions

Available scopes: `actions`, `checks`, `contents`, `deployments`, `id-token`, `issues`, `discussions`, `packages`, `pages`, `pull-requests`, `repository-projects`, `security-events`, `statuses`

Levels: `read`, `write`, `admin`

```yaml
permissions: read-all    # Read all scopes
permissions: write-all   # Write all scopes
permissions: {}          # No permissions
```

## Filter Patterns

Used in `branches`, `tags`, `paths`:

| Pattern | Matches |
|---------|---------|
| `*` | Any chars except `/` |
| `**` | Any chars including `/` |
| `?` | Zero or one preceding char |
| `[abc]` | Character class |
| `!pattern` | Negation (must be first) |

```yaml
on:
  push:
    branches: ['main', 'releases/**']
    tags: ['v[12].[0-9]+.[0-9]+']
    paths: ['src/**', '!src/**/*.test.ts']
```

Quote patterns starting with `*`, `[`, or `!` in YAML.

<!--
Source references:
- https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions
-->
