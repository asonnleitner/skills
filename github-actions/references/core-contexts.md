---
name: core-contexts
description: GitHub Actions context objects and their properties
---

# Contexts

Access context properties with dot notation (`github.sha`) or index syntax (`github['sha']`). Non-existent properties return empty string.

## `github` Context

| Property | Type | Description |
|----------|------|-------------|
| `github.actor` | string | User who triggered the workflow |
| `github.base_ref` | string | PR target branch (PR events only) |
| `github.event` | object | Full webhook payload |
| `github.event_name` | string | Triggering event name |
| `github.head_ref` | string | PR source branch (PR events only) |
| `github.job` | string | Current job ID |
| `github.ref` | string | Full ref (`refs/heads/main`, `refs/tags/v1.0`) |
| `github.ref_name` | string | Short ref name (`main`, `v1.0`) |
| `github.ref_protected` | boolean | Whether ref is protected |
| `github.ref_type` | string | `branch` or `tag` |
| `github.repository` | string | `owner/repo` |
| `github.repository_owner` | string | Repository owner |
| `github.run_id` | string | Unique workflow run ID |
| `github.run_number` | string | Sequential run number |
| `github.run_attempt` | string | Re-run attempt number (starts at 1) |
| `github.sha` | string | Triggering commit SHA |
| `github.token` | string | `GITHUB_TOKEN` value |
| `github.triggering_actor` | string | User who initiated run (differs from `actor` on re-runs) |
| `github.workflow` | string | Workflow name |
| `github.workspace` | string | Default working directory |
| `github.server_url` | string | GitHub server URL |
| `github.api_url` | string | REST API URL |

## `env` Context

Custom environment variables set at workflow/job/step level:
```yaml
env:
  MY_VAR: hello
steps:
  - run: echo "${{ env.MY_VAR }}"
```

## `vars` Context

Configuration variables from repository/organization/environment settings:
```yaml
run: echo "${{ vars.DEPLOY_TARGET }}"
```

## `job` Context

| Property | Type | Description |
|----------|------|-------------|
| `job.status` | string | `success`, `failure`, `cancelled` |
| `job.container.id` | string | Container ID |
| `job.container.network` | string | Container network ID |
| `job.services.<id>.id` | string | Service container ID |
| `job.services.<id>.ports` | object | Exposed ports |

## `steps` Context

Only steps with explicit `id` that have already completed:

| Property | Type | Description |
|----------|------|-------------|
| `steps.<id>.outputs.<name>` | string | Step output value |
| `steps.<id>.outcome` | string | Result before `continue-on-error` |
| `steps.<id>.conclusion` | string | Result after `continue-on-error` |

Values: `success`, `failure`, `cancelled`, `skipped`

```yaml
- id: build
  run: npm run build
- if: steps.build.outcome == 'success'
  run: npm run deploy
```

## `runner` Context

| Property | Values |
|----------|--------|
| `runner.os` | `Linux`, `Windows`, `macOS` |
| `runner.arch` | `X86`, `X64`, `ARM64`, `ARM` |
| `runner.temp` | Path to temp directory |
| `runner.tool_cache` | Path to tool cache |
| `runner.debug` | Debug logging enabled |

## `secrets` Context

```yaml
env:
  TOKEN: ${{ secrets.API_TOKEN }}
  GH: ${{ secrets.GITHUB_TOKEN }}  # Auto-generated
```

Not available in composite actions for security.

## `strategy` / `matrix` Context

```yaml
strategy:
  fail-fast: true       # ${{ strategy.fail-fast }}
  matrix:
    os: [ubuntu-latest, windows-latest]  # ${{ matrix.os }}
    node: [18, 20]                       # ${{ matrix.node }}

# strategy.job-index (zero-based), strategy.job-total
```

## `needs` Context

Outputs from dependent jobs:
```yaml
jobs:
  build:
    outputs:
      version: ${{ steps.ver.outputs.version }}
  deploy:
    needs: build
    steps:
      - run: echo "${{ needs.build.outputs.version }}"
      # needs.build.result → success, failure, cancelled, skipped
```

## `inputs` Context

For `workflow_dispatch` and `workflow_call`:
```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]
jobs:
  deploy:
    steps:
      - run: echo "${{ inputs.environment }}"
```

## Context Availability

| Location | Available Contexts |
|----------|-------------------|
| `run-name` | `github`, `inputs`, `vars` |
| `concurrency` | `github`, `inputs`, `vars` |
| `jobs.<id>.if` | `github`, `needs`, `vars`, `inputs` |
| `jobs.<id>.runs-on` | `github`, `needs`, `strategy`, `matrix`, `vars`, `inputs` |
| `steps[*].if` | All contexts |
| `steps[*].run` | All contexts |

<!--
Source references:
- https://docs.github.com/en/actions/reference/contexts
-->
