---
name: actions-creating
description: Creating composite, JavaScript, and Docker container custom actions
---

# Creating Custom Actions

## Composite Action

Bundle multiple steps into a reusable action. Works with any language/tool.

```
my-action/
├── action.yml
└── scripts/
    └── build.sh
```

```yaml
# action.yml
name: 'Build and Test'
inputs:
  node-version:
    description: 'Node.js version'
    default: '20'
outputs:
  test-result:
    description: 'Test outcome'
    value: ${{ steps.test.outputs.result }}

runs:
  using: 'composite'
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
    - run: npm ci
      shell: bash
    - id: test
      run: |
        npm test && echo "result=pass" >> $GITHUB_OUTPUT || echo "result=fail" >> $GITHUB_OUTPUT
      shell: bash
```

Usage in workflow:
```yaml
# From external repo
- uses: owner/my-action@v1
  with:
    node-version: '22'

# From same repo
- uses: ./.github/actions/my-action
```

### Key Points
- Every `run` step requires `shell`
- Access inputs via `INPUT_<UPPERCASE>` env vars or `${{ inputs.name }}`
- Reference action directory with `${{ github.action_path }}`
- Can nest other actions with `uses`

## JavaScript Action

Runs Node.js code directly on the runner — fastest action type.

```
my-js-action/
├── action.yml
├── src/
│   └── index.js
├── dist/
│   └── index.js      # Bundled output
├── package.json
└── rollup.config.js
```

```yaml
# action.yml
name: 'Greet and Time'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
outputs:
  time:
    description: 'Greeting time'
runs:
  using: 'node20'
  main: 'dist/index.js'
```

```javascript
// src/index.js
import * as core from '@actions/core'
import * as github from '@actions/github'

try {
  const name = core.getInput('who-to-greet')
  core.info(`Hello ${name}!`)
  core.setOutput('time', new Date().toTimeString())

  // Access webhook payload
  const payload = github.context.payload
  core.debug(`Event: ${github.context.eventName}`)
} catch (error) {
  core.setFailed(error.message)
}
```

### Toolkit API (`@actions/core`)

| Method | Purpose |
|--------|---------|
| `core.getInput(name)` | Get action input |
| `core.setOutput(name, value)` | Set action output |
| `core.setFailed(message)` | Fail action with error |
| `core.info(message)` | Log info |
| `core.warning(message)` | Log warning annotation |
| `core.error(message)` | Log error annotation |
| `core.debug(message)` | Log debug (needs ACTIONS_STEP_DEBUG) |

### Bundling

Bundle with Rollup or `@vercel/ncc` to create single file. Commit `dist/` directory, not `node_modules/`.

## Docker Container Action

Runs code in a Docker container — supports any language.

```
my-docker-action/
├── action.yml
├── Dockerfile
└── entrypoint.sh
```

```dockerfile
FROM alpine:3.19
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

```bash
#!/bin/sh -l
# entrypoint.sh
echo "Hello $1"
time=$(date)
echo "time=$time" >> $GITHUB_OUTPUT
```

```yaml
# action.yml
name: 'Hello Docker'
inputs:
  who-to-greet:
    description: 'Who to greet'
    required: true
outputs:
  time:
    description: 'Greeting time'
runs:
  using: 'docker'
  image: 'Dockerfile'
  args:
    - ${{ inputs.who-to-greet }}
```

### Key Points
- Arguments passed as positional params (`$1`, `$2`, ...)
- Exit code 0 = success, non-zero = failure
- Container path `/github/workspace` maps to `$GITHUB_WORKSPACE`
- Make scripts executable: `chmod +x` or `git update-index --chmod=+x`
- Slower than JS actions due to Docker build/startup
- Can use pre-built images: `image: 'docker://alpine:3.19'`

## Versioning

Tag releases for consumers:
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git tag -fa v1 -m "Update v1 tag"    # Move major version tag
git push origin v1.0.0 v1 --force
```

Users reference: `uses: owner/action@v1` (major) or `uses: owner/action@v1.0.0` (exact).

<!--
Source references:
- https://docs.github.com/en/actions/creating-actions/creating-a-composite-action
- https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action
- https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action
-->
