---
name: feature-matrix-strategy
description: GitHub Actions matrix strategy for running job variations across multiple configurations
---

# Matrix Strategy

Run the same job across multiple configurations. Creates a Cartesian product of all dimensions.

## Basic Matrix

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node: [18, 20, 22]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm test
```

Creates 6 jobs (2 OS x 3 Node versions). Max 256 jobs per workflow run.

## Include — Add/Extend Combinations

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    node: [18, 20]
    include:
      # Add property to all combinations
      - experimental: false
      # Override for specific combination
      - os: ubuntu-latest
        node: 22
        experimental: true
      # Add entirely new combination
      - os: macos-latest
        node: 20
```

Processing order: matrix baseline → exclude → include.

## Exclude — Remove Combinations

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    version: [18, 20, 22]
    exclude:
      - os: windows-latest
        version: 22
      - os: macos-latest
        version: 18
```

## Failure Handling

```yaml
strategy:
  fail-fast: true        # Cancel all jobs if any fails (default: true)
  max-parallel: 2        # Max concurrent jobs
  matrix:
    version: [18, 20]
    experimental: [false]
    include:
      - version: 22
        experimental: true

jobs:
  test:
    continue-on-error: ${{ matrix.experimental }}
    runs-on: ubuntu-latest
```

- `fail-fast: true` — cancel remaining jobs on first failure
- `continue-on-error: true` — job failure doesn't fail the workflow
- Combine both for experimental configurations

## Dynamic Matrix from Job Output

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set.outputs.matrix }}
    steps:
      - id: set
        run: |
          echo 'matrix={"include":[{"project":"api","node":20},{"project":"web","node":22}]}' >> "$GITHUB_OUTPUT"

  build:
    needs: setup
    strategy:
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building ${{ matrix.project }} with Node ${{ matrix.node }}"
```

Use `fromJSON()` to convert string output to matrix object.

## Single-Dimension Matrix

```yaml
strategy:
  matrix:
    version: [10, 12, 14]
runs-on: ubuntu-latest
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.version }}
```

## Context from Repository Dispatch

```yaml
on:
  repository_dispatch:
    types: [test]
jobs:
  test:
    strategy:
      matrix:
        version: ${{ github.event.client_payload.versions }}
```

Trigger with API payload: `{"event_type": "test", "client_payload": {"versions": [18, 20, 22]}}`

<!--
Source references:
- https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix
- https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs
-->
