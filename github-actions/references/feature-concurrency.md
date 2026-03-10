---
name: feature-concurrency
description: GitHub Actions concurrency control and workflow cancellation process
---

# Concurrency

Control parallel execution of workflows and jobs using concurrency groups.

## Configuration

```yaml
# Workflow level — one deploy at a time
concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: true

# Job level
jobs:
  deploy:
    concurrency:
      group: deploy-production
      cancel-in-progress: false
```

- `group` — name identifying the concurrency group (supports expressions)
- `cancel-in-progress` — cancel currently running job/workflow in the same group when a new one is queued

## Common Patterns

```yaml
# Cancel outdated PR checks
concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# Never cancel production deploys
concurrency:
  group: deploy-production
  cancel-in-progress: false

# Per-environment concurrency
concurrency:
  group: deploy-${{ inputs.environment }}
  cancel-in-progress: ${{ inputs.environment != 'production' }}
```

## Cancellation Process

When a workflow run is cancelled (manually or via `cancel-in-progress`):

1. **Re-evaluate `if` conditions** — jobs with `if: always()` continue running
2. **Send cancellation message** to all runner machines
3. **Re-evaluate step `if` conditions** — steps with `if: cancelled()` or `if: always()` execute
4. **Signal sequence** for running steps:
   - `SIGINT` → 7.5 seconds grace period
   - `SIGTERM` → 2.5 seconds grace period
   - Force kill process tree
5. **Global timeout**: 5 minutes after cancellation, all remaining jobs/steps force-terminated

## Handling Cancellation in Scripts

```yaml
- name: Cleanup
  if: cancelled()
  run: echo "Workflow was cancelled, cleaning up..."

# Prefer !cancelled() over always() for safer execution
- name: Report status
  if: '!cancelled()'
  run: echo "Runs on success or failure, but not on cancellation"
```

## Key Behaviors

- Default: multiple workflow runs execute concurrently
- Concurrency groups are shared across workflows in the same repository
- `cancel-in-progress` only cancels runs in the **same concurrency group**
- Queued runs wait (up to limit) when another run with the same group is in progress
- Avoid matching `concurrency.group` between caller and reusable workflows

<!--
Source references:
- https://docs.github.com/en/actions/using-jobs/using-concurrency
- https://docs.github.com/en/actions/reference/workflow-cancellation
-->
