---
name: feature-workflow-commands
description: GitHub Actions workflow commands for outputs, env vars, paths, summaries, and annotations
---

# Workflow Commands

## Setting Outputs (`GITHUB_OUTPUT`)

Set step outputs accessible by later steps/jobs. Step must have `id`.

```yaml
- name: Set version
  id: version
  run: echo "value=1.2.3" >> "$GITHUB_OUTPUT"

- name: Use version
  run: echo "${{ steps.version.outputs.value }}"
```

Multiline output:
```bash
{
  echo 'json<<EOF'
  cat result.json
  echo EOF
} >> "$GITHUB_OUTPUT"
```

## Setting Environment Variables (`GITHUB_ENV`)

Available to all **subsequent** steps in the job (not the current step).

```bash
echo "NODE_ENV=production" >> "$GITHUB_ENV"
```

Multiline value:
```bash
{
  echo 'BODY<<EOF'
  echo "line1"
  echo "line2"
  echo EOF
} >> "$GITHUB_ENV"
```

## Adding to System PATH (`GITHUB_PATH`)

Prepends directory to `PATH` for subsequent steps:
```bash
echo "$HOME/.local/bin" >> "$GITHUB_PATH"
```

## Job Summaries (`GITHUB_STEP_SUMMARY`)

Add GitHub Flavored Markdown to workflow run summary page:
```bash
echo "### Build Results :rocket:" >> $GITHUB_STEP_SUMMARY
echo "" >> $GITHUB_STEP_SUMMARY
echo "| Test | Result |" >> $GITHUB_STEP_SUMMARY
echo "|------|--------|" >> $GITHUB_STEP_SUMMARY
echo "| Unit | Passed |" >> $GITHUB_STEP_SUMMARY
```

- Max 1 MiB per step
- Overwrite with `>` instead of `>>`
- Delete with `rm $GITHUB_STEP_SUMMARY`

## Annotations

```bash
echo "::notice file=app.js,line=1,col=5::Missing semicolon"
echo "::warning file=app.js,line=10::Deprecated API usage"
echo "::error file=app.js,line=20,title=Syntax Error::Unexpected token"
```

Parameters: `file`, `line`, `endLine`, `col`, `endColumn`, `title`

Debug messages (visible only with `ACTIONS_STEP_DEBUG` secret set to `true`):
```bash
echo "::debug::Diagnostic info"
```

## Log Grouping

```bash
echo "::group::Install Dependencies"
npm ci
echo "::endgroup::"
```

## Masking Values

Prevent values from appearing in logs:
```bash
echo "::add-mask::$SECRET_VALUE"
```

- Register masks **before** outputting sensitive values
- Masked values cannot be set as outputs
- Each word in masked value is individually redacted

```yaml
- name: Generate and mask secret
  id: gen
  run: |
    TOKEN=$(openssl rand -hex 16)
    echo "::add-mask::$TOKEN"
    echo "token=$TOKEN" >> "$GITHUB_OUTPUT"
```

## Stop/Resume Commands

Temporarily disable workflow command processing:
```bash
STOP=$(uuidgen)
echo "::stop-commands::$STOP"
echo "::warning::This won't render as a warning"
echo "::$STOP::"
echo "::warning::This will render as a warning"
```

## PowerShell Equivalents

```powershell
"MY_VAR=value" >> $env:GITHUB_ENV
"output_name=value" >> $env:GITHUB_OUTPUT
"C:\tools\bin" >> $env:GITHUB_PATH
"### Summary" >> $env:GITHUB_STEP_SUMMARY
```

For PowerShell 5.1 and below, use `Out-File -Encoding utf8 -Append`.

<!--
Source references:
- https://docs.github.com/en/actions/reference/workflow-commands-for-github-actions
-->
