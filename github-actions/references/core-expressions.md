---
name: core-expressions
description: GitHub Actions expression syntax, operators, functions, and status checks
---

# Expressions

Expressions use `${{ <expression> }}` syntax. In `if` conditionals, `${{ }}` is optional.

## Literals

- **boolean**: `true`, `false`
- **null**: `null`
- **number**: `123`, `0xff`, `-2.99e-2`
- **string**: Single-quoted (`'hello'`), escape `'` with `''`. Double quotes throw errors.

## Operators

| Operator | Description |
|----------|-------------|
| `( )` | Grouping |
| `[ ]` | Index access |
| `.` | Property access |
| `!` | Not |
| `&&`, `||` | And, Or |
| `==`, `!=` | Equality (loose, case-insensitive for strings) |
| `<`, `<=`, `>`, `>=` | Comparison |

**Type coercion** (loose equality): `null` → `0`, `true` → `1`, `false` → `0`, empty string → `0`. NaN comparisons always return `false`.

## Functions

### `contains(search, item)`
Case-insensitive. Works on strings and arrays.
```yaml
if: contains(github.event.issue.labels.*.name, 'bug')
if: contains(fromJSON('["push", "pull_request"]'), github.event_name)
```

### `startsWith(string, value)` / `endsWith(string, value)`
Case-insensitive string matching.
```yaml
if: startsWith(github.ref, 'refs/tags/')
```

### `format(string, ...values)`
Replace `{N}` placeholders. Escape braces with `{{` and `}}`.
```yaml
run: echo "${{ format('Hello {0} {1}!', 'Mona', 'Octocat') }}"
```

### `join(array, separator)`
Concatenate array to string. Default separator: `,`.
```yaml
run: echo "${{ join(github.event.issue.labels.*.name, ', ') }}"
```

### `toJSON(value)` / `fromJSON(value)`
Serialize/deserialize JSON. Essential for passing structured data between jobs.
```yaml
# Convert string output to boolean
if: fromJSON(steps.check.outputs.should-deploy)

# Dynamic matrix from job output
strategy:
  matrix:
    version: ${{ fromJSON(needs.setup.outputs.versions) }}
```

### `hashFiles(...patterns)`
SHA-256 hash of files matching glob patterns. Relative to `GITHUB_WORKSPACE`.
```yaml
key: npm-${{ hashFiles('**/package-lock.json') }}
key: deps-${{ hashFiles('**/package-lock.json', '**/Gemfile.lock') }}
key: go-${{ hashFiles('**/go.sum', '!vendor/**') }}
```

### `case(pred1, val1, ..., default)`
Conditional value selection.
```yaml
env:
  DEPLOY_ENV: ${{ case(
    github.ref == 'refs/heads/main', 'production',
    github.ref == 'refs/heads/staging', 'staging',
    'development'
  ) }}
```

## Status Check Functions

Used in `if` conditionals. Default implicit check is `success()`.

| Function | Returns `true` when |
|----------|---------------------|
| `success()` | All previous steps succeeded (default) |
| `failure()` | Any previous step failed |
| `cancelled()` | Workflow was cancelled |
| `always()` | Always, even when cancelled |

```yaml
# Run cleanup even on failure
- if: always()
  run: cleanup.sh

# Run only on failure of a specific step
- if: failure() && steps.build.conclusion == 'failure'
  run: echo "Build failed"

# Prefer !cancelled() over always() for safer execution
- if: '!cancelled()'
  run: echo "Runs unless cancelled"
```

**Important:** When combining status functions with conditions, always include the status function explicitly:
```yaml
# Wrong - success() is implicit, won't run on failure
- if: github.ref == 'refs/heads/main'

# Right - explicitly handle failure case
- if: failure() && github.ref == 'refs/heads/main'
```

## Object Filters

Extract values from collections using `*.property`:
```yaml
# Get all label names from an issue
github.event.issue.labels.*.name  # → ["bug", "help wanted"]
```

<!--
Source references:
- https://docs.github.com/en/actions/reference/expressions
-->
