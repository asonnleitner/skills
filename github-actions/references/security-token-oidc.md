---
name: security-token-oidc
description: GITHUB_TOKEN permissions, lifetime, and OpenID Connect for cloud providers
---

# GITHUB_TOKEN

Automatically created per job for authenticating with GitHub APIs.

## Permissions

```yaml
permissions:
  contents: read
  pull-requests: write
  packages: write
  id-token: write        # Required for OIDC
```

Available scopes: `actions`, `checks`, `contents`, `deployments`, `id-token`, `issues`, `discussions`, `packages`, `pages`, `pull-requests`, `repository-projects`, `security-events`, `statuses`

Set at workflow or job level. Job-level overrides workflow-level.

**Fork PRs**: Write tokens automatically downgraded to read.

## Lifetime

- **GitHub-hosted runners**: max 6 hours (job limit)
- **Self-hosted runners**: max 24 hours token refresh, 5 days job limit
- Expires when job completes

## Key Behaviors

- Actions triggered by `GITHUB_TOKEN` do NOT trigger additional workflows (prevents infinite loops)
- Scoped to the repository containing the workflow
- In reusable workflows, permissions can only be downgraded (never elevated)

---

# OpenID Connect (OIDC)

Request short-lived tokens from cloud providers without storing long-lived credentials.

## Setup

```yaml
permissions:
  id-token: write    # Required
  contents: read

steps:
  - uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: arn:aws:iam::123456:role/deploy
      aws-region: us-east-1
```

## Token Claims

| Claim | Example | Use For |
|-------|---------|---------|
| `sub` | `repo:org/repo:ref:refs/heads/main` | Primary trust policy |
| `iss` | `https://token.actions.githubusercontent.com` | Token issuer |
| `aud` | `https://github.com/org` | Audience |
| `repository` | `org/repo` | Repository filter |
| `repository_owner` | `org` | Organization filter |
| `ref` | `refs/heads/main` | Branch/tag filter |
| `environment` | `production` | Environment filter |
| `event_name` | `push` | Event filter |
| `job_workflow_ref` | `org/repo/.github/workflows/deploy.yml@refs/heads/main` | Reusable workflow filter |
| `runner_environment` | `github-hosted` | Runner type filter |
| `repository_visibility` | `public` | Visibility filter |

## Subject Claim Formats

Default `sub` format depends on context:
- **Branch**: `repo:ORG/REPO:ref:refs/heads/BRANCH`
- **Tag**: `repo:ORG/REPO:ref:refs/tags/TAG`
- **Environment**: `repo:ORG/REPO:environment:ENV_NAME`
- **Pull request**: `repo:ORG/REPO:pull_request`

Colons in metadata replaced with `%3A`.

## Cloud Provider Configuration

### AWS
```json
{
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:sub": "repo:org/repo:ref:refs/heads/main",
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
    }
  }
}
```

### Azure
```
repo:org/repo:ref:refs/heads/main
```

### GCP
```
assertion.sub == 'repo:org/repo:ref:refs/heads/main'
```

## Manual Token Request

```bash
curl -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
  "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=api://AzureADTokenExchange"
```

Environment variables `ACTIONS_ID_TOKEN_REQUEST_URL` and `ACTIONS_ID_TOKEN_REQUEST_TOKEN` are available when `id-token: write` is set.

## Customizing Claims

Use REST API to configure custom `sub` format:
- `["repository_owner"]` — allow any repo in org
- `["repository_id"]` — rename-safe repository filter
- `["job_workflow_ref"]` — require specific reusable workflow

**Critical**: Always define at least one condition in your cloud provider to prevent untrusted repos from requesting tokens.

<!--
Source references:
- https://docs.github.com/en/actions/security-guides/automatic-token-authentication
- https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect
-->
