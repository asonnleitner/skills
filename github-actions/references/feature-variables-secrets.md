---
name: feature-variables-secrets
description: GitHub Actions variables, secrets, default environment variables, and their usage patterns
---

# Variables & Secrets

## Default Environment Variables

Always available in every step (no `${{ }}` needed):

| Variable | Description |
|----------|-------------|
| `CI` | Always `true` |
| `GITHUB_ACTIONS` | Always `true` |
| `GITHUB_ACTOR` | User who triggered the workflow |
| `GITHUB_REPOSITORY` | `owner/repo` |
| `GITHUB_REF` | Full ref (`refs/heads/main`) |
| `GITHUB_REF_NAME` | Short ref (`main`) |
| `GITHUB_SHA` | Triggering commit SHA |
| `GITHUB_WORKFLOW` | Workflow name |
| `GITHUB_RUN_ID` | Unique run ID |
| `GITHUB_RUN_NUMBER` | Sequential run number |
| `GITHUB_EVENT_NAME` | Triggering event name |
| `GITHUB_WORKSPACE` | Repository checkout path |
| `GITHUB_BASE_REF` | PR target branch (PR events only) |
| `GITHUB_HEAD_REF` | PR source branch (PR events only) |
| `RUNNER_OS` | `Linux`, `Windows`, `macOS` |
| `RUNNER_ARCH` | Processor architecture |
| `RUNNER_TEMP` | Temp directory path |

## Custom Variables

Set via repository/organization/environment settings UI. Access with `vars` context:

```yaml
steps:
  - run: echo "Deploying to ${{ vars.DEPLOY_TARGET }}"
    env:
      API_URL: ${{ vars.API_URL }}
```

### Limits

| Scope | Max Count | Max Size |
|-------|-----------|----------|
| Organization | 1,000 | Combined 256 KB per run |
| Repository | 500 | Combined 256 KB per run |
| Environment | 100 | Not counted in size limit |
| Per variable | — | 48 KB |

## Secrets

Access with `secrets` context. Automatically redacted in logs:

```yaml
steps:
  - run: deploy --token "$TOKEN"
    env:
      TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

### Limits

| Scope | Max Count |
|-------|-----------|
| Organization | 1,000 |
| Repository | 100 |
| Environment | 100 |
| Per secret | 48 KB |

### Best Practices

- Don't store structured data (JSON/XML) as secrets — use individual secrets per value
- Use `::add-mask::` for dynamically generated sensitive values
- Prefer GitHub Apps or fine-grained PATs over broad-scope tokens
- Environment secrets with required reviewers add approval gates

## Precedence

**Environment > Repository > Organization** (for both variables and secrets)

When names collide, the more specific scope wins.

## Workflow-Level Environment Variables

```yaml
env:
  NODE_ENV: production        # Available to all jobs/steps

jobs:
  build:
    env:
      BUILD_TYPE: release     # Available to all steps in this job
    steps:
      - env:
          VERBOSE: "true"     # Available to this step only
        run: build.sh
```

- Step env overrides job env overrides workflow env
- Cannot reference other env vars in the same `env` map
- In reusable workflows, caller's workflow-level `env` is NOT propagated

## Using Variables in Different Contexts

```yaml
# In expressions
if: vars.USE_FEATURE == 'true'

# In run commands via env
env:
  TOKEN: ${{ secrets.TOKEN }}
run: curl -H "Authorization: $TOKEN" $API_URL

# In action inputs
uses: actions/setup-node@v4
with:
  node-version: ${{ vars.NODE_VERSION }}
```

<!--
Source references:
- https://docs.github.com/en/actions/reference/variables
- https://docs.github.com/en/actions/reference/encrypted-secrets
-->
