---
name: actions-metadata
description: GitHub Actions action.yml metadata syntax for custom actions
---

# Action Metadata (`action.yml`)

Every custom action requires an `action.yml` (or `action.yaml`) at the repository root or action directory.

## Inputs & Outputs

```yaml
name: 'My Action'
description: 'Does something useful'
author: 'octocat'

inputs:
  api-key:
    description: 'API key for authentication'
    required: true
  version:
    description: 'Version to deploy'
    required: false
    default: 'latest'
  old-param:
    description: 'Deprecated'
    deprecationMessage: 'Use version instead'

outputs:
  result:
    description: 'Deployment result URL'
```

- Input IDs become `INPUT_<UPPERCASE_ID>` env vars (spaces → `_`)
- IDs must start with letter or `_`, contain only alphanumeric, `-`, `_`
- `required: true` is informational only — doesn't auto-error
- Outputs not declared can still be set via workflow commands

## JavaScript Action

```yaml
runs:
  using: 'node20'      # or 'node24'
  main: 'dist/index.js'
  pre: 'setup.js'      # Runs at job start (optional)
  pre-if: runner.os == 'linux'  # Condition for pre (default: always())
  post: 'cleanup.js'   # Runs at job end (optional)
  post-if: runner.os == 'linux' # Condition for post (default: always())
```

## Composite Action

```yaml
runs:
  using: 'composite'
  steps:
    - name: Install
      run: npm ci
      shell: bash                    # Required with run
      working-directory: ./app
    - name: Build
      id: build
      run: |
        echo "hash=$(sha256sum dist/bundle.js | cut -d' ' -f1)" >> $GITHUB_OUTPUT
      shell: bash
    - name: Deploy
      uses: actions/deploy@v1       # Can use other actions
      with:
        target: production
      if: github.ref == 'refs/heads/main'
      continue-on-error: true

outputs:
  build-hash:
    description: 'Build hash'
    value: ${{ steps.build.outputs.hash }}  # Required for composite
```

- `shell` is required for every `run` step
- Reference action directory: `${{ github.action_path }}` or `$GITHUB_ACTION_PATH`
- Composite outputs must use `value` mapping to step outputs

## Docker Action

```yaml
runs:
  using: 'docker'
  image: 'Dockerfile'              # Local Dockerfile or 'docker://alpine:3.8'
  env:
    MY_VAR: 'value'
  entrypoint: '/entrypoint.sh'     # Override ENTRYPOINT (optional)
  args:                            # Passed to ENTRYPOINT
    - ${{ inputs.greeting }}
    - 'world'
  pre-entrypoint: 'setup.sh'      # Before main (optional)
  pre-if: always()
  post-entrypoint: 'cleanup.sh'   # After main (optional)
  post-if: always()
```

- Image options: `Dockerfile` (local), `docker://image:tag` (registry)
- `args` replaces Docker CMD instruction
- Pre/post entrypoints run in separate containers with same base image

## Branding

```yaml
branding:
  icon: 'award'         # Feather icon name (feathericons.com)
  color: 'green'        # white, black, yellow, blue, green, orange, red, purple, gray-dark
```

<!--
Source references:
- https://docs.github.com/en/actions/creating-actions/metadata-syntax-for-github-actions
-->
