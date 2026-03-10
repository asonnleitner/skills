---
name: security-best-practices
description: GitHub Actions security best practices and usage limits
---

# Security Best Practices

## Script Injection Prevention

Never interpolate untrusted context directly in `run` commands:

```yaml
# DANGEROUS - user-controlled title injected into shell
- run: echo "Title: ${{ github.event.issue.title }}"

# SAFE - use intermediate environment variable
- env:
    TITLE: ${{ github.event.issue.title }}
  run: echo "Title: $TITLE"

# SAFEST - use an action instead of inline script
- uses: my-org/process-title@v1
  with:
    title: ${{ github.event.issue.title }}
```

Vulnerable contexts include: `github.event.issue.title`, `github.event.pull_request.body`, `github.event.comment.body`, `github.head_ref`, and any user-controlled event data.

## Third-Party Action Security

```yaml
# Pin to full commit SHA (immutable)
- uses: actions/checkout@8f4b7f84864484a7bf31766abe9204da3cbe65b3

# Tag references can be moved (less secure)
- uses: actions/checkout@v4
```

- Always pin to full-length commit SHA for production workflows
- Audit action source code — verify secrets aren't leaked
- Use Dependabot to keep pinned actions updated
- Repository-level policies can enforce SHA-pinning

## Secret Management

- **Least privilege**: only grant minimum required permissions
- **No structured data**: don't use JSON/XML as secret values — use separate secrets per value
- **Rotate regularly**: reduce exposure window of compromised credentials
- **Mask generated values**: use `::add-mask::` for dynamically created sensitive data
- **Environment secrets with reviewers**: add approval gates for sensitive environments
- **Prefer GitHub Apps**: fine-grained permissions, short-lived tokens, not tied to individual users

## Self-Hosted Runner Security

- **Never use for public repositories** — any fork PR can run code on your runner
- Self-hosted runners don't provide ephemeral, clean environments like GitHub-hosted runners
- Use **Just-In-Time (JIT) runners** for ephemeral, single-job execution
- **Runner groups**: isolate runners by organization/repository
- Minimize sensitive data and network access on runner machines

## Untrusted Code

- Avoid `pull_request_target` with code checkout from fork PRs
- Never run `npm install` or build commands on untrusted PR code in privileged context
- Use `pull_request` (not `pull_request_target`) when possible
- Use CodeQL and OpenSSF Scorecard to detect vulnerable workflows

## Permissions Best Practice

```yaml
# Restrict to minimum needed
permissions:
  contents: read
  pull-requests: write

# Or remove all by default, grant per job
permissions: {}

jobs:
  build:
    permissions:
      contents: read
  deploy:
    permissions:
      contents: read
      id-token: write
```

---

# Usage Limits

## Workflow Limits

| Limit | Value |
|-------|-------|
| Workflow run time | 35 days max |
| Job execution (GitHub-hosted) | 6 hours |
| Job execution (self-hosted) | 5 days |
| Job queue time | 24 hours |
| Matrix jobs per run | 256 |
| Workflow trigger events | 1,500 per 10s per repo |
| Queued workflow runs | 500 per 10s |

## Concurrency Limits (GitHub-hosted)

| Plan | Total Concurrent | macOS Max |
|------|-----------------|-----------|
| Free | 20 | 5 |
| Pro | 40 | 5 |
| Team | 60 | 5 |
| Enterprise | 500 | 50 |

## Cache Limits

| Limit | Value |
|-------|-------|
| Per repository | 10 GB default |
| Upload rate | 200/min per repo |
| Download rate | 1,500/min per repo |
| Retention | 7 days without access |

## Docker Hub Rate Limits

- GitHub-hosted pulling **public** images: not rate-limited
- GitHub-hosted pulling **private** images: rate-limited
- Self-hosted: always rate-limited

<!--
Source references:
- https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions
- https://docs.github.com/en/actions/reference/usage-limits-billing-and-administration
-->
