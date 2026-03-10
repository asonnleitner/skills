---
name: core-events
description: GitHub Actions workflow trigger events, filters, and activity types
---

# Events & Triggers

## Push & Pull Request

```yaml
on:
  push:
    branches: [main, 'releases/**']
    branches-ignore: ['test/**']
    tags: ['v*']
    paths: ['src/**', '!docs/**']

  pull_request:
    # Default types: opened, synchronize, reopened
    types: [opened, synchronize, reopened, ready_for_review]
    branches: [main]
    paths: ['**.ts', '**.tsx']
```

- Cannot use `branches` and `branches-ignore` together (same for `tags`, `paths`)
- `pull_request` runs on merge commit ref (`refs/pull/N/merge`)
- `pull_request` doesn't run on merge conflicts

### `pull_request_target`

Runs in **base branch** context (safe for fork PRs). Use for labeling/commenting on fork PRs without exposing secrets to untrusted code:
```yaml
on:
  pull_request_target:
    types: [opened, labeled]
```

## Manual Triggers

### `workflow_dispatch`

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy target'
        required: true
        type: choice
        options: [staging, production]
      debug:
        description: 'Enable debug'
        type: boolean
        default: false
      version:
        description: 'Version number'
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying ${{ inputs.version }} to ${{ inputs.environment }}"
```

Input types: `string`, `boolean`, `choice`, `number`, `environment`

### `repository_dispatch`

API-triggered event for external automation:
```yaml
on:
  repository_dispatch:
    types: [deploy, rollback]

jobs:
  handle:
    steps:
      - run: echo "${{ github.event.client_payload.ref }}"
```

Trigger via API with `event_type` and `client_payload` (max 65,535 chars, 10 top-level properties).

## Schedule

```yaml
on:
  schedule:
    - cron: '0 6 * * 1-5'   # Weekdays at 6:00 UTC
    - cron: '15 */4 * * *'   # Every 4 hours at :15
```

- Format: `minute hour day-of-month month day-of-week`
- Runs on default branch only
- Disabled after 60 days of repo inactivity (public repos)
- May have slight execution delay
- `@daily`, `@weekly` etc. NOT supported

## Reusable Workflow

```yaml
on:
  workflow_call:
    inputs:
      config:
        type: string
        required: true
    outputs:
      result:
        description: 'Build result'
        value: ${{ jobs.build.outputs.result }}
    secrets:
      token:
        required: true
```

## Workflow Chaining

```yaml
on:
  workflow_run:
    workflows: [CI, Build]
    types: [completed]
    branches: [main]

jobs:
  deploy:
    if: github.event.workflow_run.conclusion == 'success'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying after successful CI"
```

- Access secrets and write tokens (unlike `pull_request` from forks)
- Max 3 levels of chaining

## Common Webhook Events

| Event | Default Activity Types | Key Use |
|-------|----------------------|---------|
| `push` | n/a | Code pushed |
| `pull_request` | opened, synchronize, reopened | PR activity |
| `issues` | opened, edited, closed, labeled, ... | Issue management |
| `issue_comment` | created, edited, deleted | Issue/PR comments |
| `release` | published, created, edited, ... | Release management |
| `create` | n/a | Branch/tag created |
| `delete` | n/a | Branch/tag deleted |
| `merge_group` | checks_requested | Merge queue |
| `discussion` | created, edited, answered, ... | Discussions |
| `fork` | n/a | Repository forked |
| `watch` | started | Repository starred |

### Filtering by Activity Type

```yaml
on:
  issues:
    types: [opened, labeled]
  pull_request_review:
    types: [submitted]
```

## Key Behaviors

- Most webhook events run on the **default branch** for `GITHUB_SHA`/`GITHUB_REF`
- `push` context: tip commit of pushed ref
- `pull_request` context: last merge commit (`refs/pull/N/merge`)
- Bulk operations (3+ tags created/deleted) may not trigger events
- Workflows triggered by `GITHUB_TOKEN` don't trigger additional workflows (prevents loops)

<!--
Source references:
- https://docs.github.com/en/actions/reference/events-that-trigger-workflows
-->
