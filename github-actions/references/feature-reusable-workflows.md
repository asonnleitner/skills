---
name: feature-reusable-workflows
description: GitHub Actions reusable workflows with workflow_call, inputs, outputs, and secrets
---

# Reusable Workflows

Define a workflow once and call it from other workflows.

## Defining a Reusable Workflow

```yaml
# .github/workflows/build.yml
name: Build
on:
  workflow_call:
    inputs:
      environment:
        type: string
        required: true
      node-version:
        type: number
        default: 20
    outputs:
      artifact-url:
        description: 'URL of built artifact'
        value: ${{ jobs.build.outputs.url }}
    secrets:
      deploy-token:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      url: ${{ steps.upload.outputs.url }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm run build
        env:
          TOKEN: ${{ secrets.deploy-token }}
      - id: upload
        run: echo "url=https://example.com/artifact" >> "$GITHUB_OUTPUT"
```

Input types: `string`, `number`, `boolean`

## Calling a Reusable Workflow

```yaml
jobs:
  build-staging:
    uses: org/repo/.github/workflows/build.yml@main
    with:
      environment: staging
      node-version: 22
    secrets:
      deploy-token: ${{ secrets.STAGING_TOKEN }}

  # Inherit all caller secrets
  build-prod:
    uses: org/repo/.github/workflows/build.yml@v1
    with:
      environment: production
    secrets: inherit

  # Local reusable workflow
  lint:
    uses: ./.github/workflows/lint.yml

  # Use outputs from reusable workflow
  deploy:
    needs: build-staging
    runs-on: ubuntu-latest
    steps:
      - run: echo "${{ needs.build-staging.outputs.artifact-url }}"
```

## Caller Job Restrictions

Jobs calling reusable workflows only support these keys:

- `uses`, `with`, `secrets`
- `name`, `needs`, `if`
- `permissions`, `concurrency`, `strategy`

No `steps`, `runs-on`, `container`, or `services` alongside `uses`.

## Access Rules

| Called Workflow Location | Accessible From |
|-------------------------|-----------------|
| Same repository | Always |
| Public repository | Any repository (with policy) |
| Internal repository | Same org (with access settings) |
| Private repository | Same org (with access settings) |

## Nesting Limits

| Plan | Max Nesting Depth | Max Unique Workflows |
|------|-------------------|---------------------|
| Free/Enterprise Cloud | 10 levels | 50 per workflow file |
| Enterprise Server | 4 levels | 20 per workflow file |

## Key Behaviors

- **Environment variables**: Caller's workflow-level `env` is NOT propagated to called workflow. Use `vars` context or outputs instead
- **Permissions**: `GITHUB_TOKEN` can only be downgraded (not elevated) by called workflow
- **`github` context**: Always associated with caller workflow
- **Runners**: Called workflow can access caller's self-hosted runners (same org/enterprise)
- **Concurrency**: Avoid same `concurrency.group` in caller and called workflow to prevent mutual cancellation

<!--
Source references:
- https://docs.github.com/en/actions/using-workflows/reusing-workflows
-->
