---
name: github-actions
description: GitHub Actions workflow syntax, expressions, contexts, events, custom actions, caching, matrix strategies, reusable workflows, security, and best practices. Use when writing or debugging CI/CD workflows in `.github/workflows/`.
metadata:
  author: Andreas Sonnleitner
  version: "2026.3.10"
  source: Generated from https://github.com/github/docs, scripts located at https://github.com/asonnleitner/skills
---

> Based on GitHub Actions documentation as of March 2026.

GitHub Actions automates CI/CD workflows directly in GitHub repositories. Workflows are YAML files in `.github/workflows/` triggered by events like push, pull request, schedule, or manual dispatch.

## Core

| Topic | Description | Reference |
|-------|-------------|-----------|
| Workflow Syntax | Complete YAML syntax: `name`, `on`, `jobs`, `steps`, `runs-on`, `permissions`, `defaults`, `timeout-minutes` | [core-workflow-syntax](references/core-workflow-syntax.md) |
| Expressions | Operators, functions (`contains`, `startsWith`, `format`, `fromJSON`, `hashFiles`), status checks, type coercion | [core-expressions](references/core-expressions.md) |
| Contexts | `github`, `env`, `vars`, `job`, `steps`, `runner`, `secrets`, `matrix`, `needs`, `inputs` context objects | [core-contexts](references/core-contexts.md) |
| Events & Triggers | `push`, `pull_request`, `schedule`, `workflow_dispatch`, `workflow_call`, `workflow_run`, filters | [core-events](references/core-events.md) |

## Features

| Topic | Description | Reference |
|-------|-------------|-----------|
| Workflow Commands | `GITHUB_OUTPUT`, `GITHUB_ENV`, `GITHUB_PATH`, `GITHUB_STEP_SUMMARY`, annotations, masking | [feature-workflow-commands](references/feature-workflow-commands.md) |
| Variables & Secrets | Default env vars, custom variables, secrets, naming, limits, precedence | [feature-variables-secrets](references/feature-variables-secrets.md) |
| Matrix Strategy | Multi-dimensional job variations, `include`/`exclude`, `fail-fast`, `max-parallel`, dynamic matrices | [feature-matrix-strategy](references/feature-matrix-strategy.md) |
| Dependency Caching | `actions/cache`, `setup-*` auto-caching, cache keys, restore keys, eviction, limits | [feature-caching](references/feature-caching.md) |
| Reusable Workflows | `workflow_call`, inputs/outputs/secrets, nesting limits, access rules, `inherit` | [feature-reusable-workflows](references/feature-reusable-workflows.md) |
| Concurrency | Concurrency groups, `cancel-in-progress`, cancellation signals, cleanup timing | [feature-concurrency](references/feature-concurrency.md) |
| Artifacts & Environments | `upload-artifact`/`download-artifact`, deployment environments, protection rules | [feature-artifacts-environments](references/feature-artifacts-environments.md) |
| Containers & Services | `container`, `services`, Docker images, registry credentials, networking, volumes | [feature-containers](references/feature-containers.md) |

## Custom Actions

| Topic | Description | Reference |
|-------|-------------|-----------|
| Action Metadata | `action.yml` syntax: inputs, outputs, `runs` (node20/composite/docker), branding | [actions-metadata](references/actions-metadata.md) |
| Creating Actions | Composite, JavaScript (Node.js), and Docker container action patterns | [actions-creating](references/actions-creating.md) |

## Security & Limits

| Topic | Description | Reference |
|-------|-------------|-----------|
| GITHUB_TOKEN & OIDC | Token permissions, lifetime, OIDC claims, cloud provider integration | [security-token-oidc](references/security-token-oidc.md) |
| Security Best Practices | Script injection, secret management, third-party actions, self-hosted runners, limits | [security-best-practices](references/security-best-practices.md) |
