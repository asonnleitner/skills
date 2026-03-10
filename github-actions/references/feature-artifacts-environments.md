---
name: feature-artifacts-environments
description: GitHub Actions workflow artifacts and deployment environments
---

# Artifacts

Persist data between jobs and after workflow completion.

## Upload & Download

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
      - run: deploy.sh dist/
```

## Common Artifact Types

- Build outputs (binaries, bundles)
- Test results and coverage reports
- Log files and screenshots
- Generated documentation

## Key Points

- Artifacts are distinct from dependency caching — caching speeds up builds, artifacts persist outputs
- Default retention follows repository settings
- Artifacts from deleted workflow runs are removed per retention policy
- Use `actions/upload-artifact@v4` / `actions/download-artifact@v4`

---

# Deployment Environments

Configure environments with protection rules, secrets, and deployment tracking.

## Configuration

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - run: deploy.sh
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}  # Environment-scoped secret
```

## Protection Rules

- **Required reviewers** — specific people must approve before job runs
- **Wait timer** — delay before job proceeds
- **Deployment branches** — restrict which branches can deploy
- **Custom protection rules** — webhook-based approval gates

## Key Behaviors

- Each job references a single environment
- Protection rules must **all pass** before job is sent to a runner
- Environment secrets only available **after** protection rules pass
- Deployments appear in repository's deployment history
- Environment-level variables/secrets override repository-level ones

## Staged Deployment Pattern

```yaml
jobs:
  deploy-staging:
    environment: staging
    runs-on: ubuntu-latest
    steps:
      - run: deploy.sh staging

  deploy-production:
    needs: deploy-staging
    environment:
      name: production
      url: https://myapp.com
    runs-on: ubuntu-latest
    steps:
      - run: deploy.sh production
```

<!--
Source references:
- https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
- https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment
-->
