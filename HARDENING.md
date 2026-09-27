<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tags rather than full 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (tag `v3`, marked `# ratchet:exclude`)
- `actions/upload-artifact@v6` (tag `v6`, marked `# ratchet:exclude`)

The third reference `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly SHA-pinned.

Locations:

- `action.yml:218`
- `action.yml:430`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes `$SANITIZED` to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). `SANITIZED` is derived from the `WORKFLOW_NAME` env var, which holds `${{ inputs.workflow_name }}` (defaulting to `${{ github.workflow }}`). While the `sed | xargs | tr` pipeline does strip some whitespace, it does not apply the required `tr -d '\n\r'` sanitization immediately before the write. An attacker-controlled `workflow_name` input containing embedded newlines could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

Offending line: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:197`

### script-injection (severity: high)

Rule (b) violation: The 'Upload Telemetry to Google Cloud' step expands env vars holding workflow-controllable values inside double-quoted shell strings passed to `sed`, allowing bash command substitution (`$(...)`) to execute:

```
sed -e "s#OTLP_GOOGLE_CLOUD_PROJECT#${OTLP_GOOGLE_CLOUD_PROJECT}#g" \\
    -e "s#GITHUB_REPOSITORY_PLACEHOLDER#${GITHUB_REPOSITORY}#g" \\
    -e "s#GITHUB_RUN_ID_PLACEHOLDER#${GITHUB_RUN_ID}#g" \\
```

`${OTLP_GOOGLE_CLOUD_PROJECT}` is set from `inputs.gcp_project_id` (attacker-controlled). `${GITHUB_REPOSITORY}` and `${GITHUB_RUN_ID}` are set from `github.repository` and `github.run_id` (GitHub-controlled but still flow through YAML template substitution). A value like `$(malicious_command)` in any of these variables would be executed by bash inside the double-quoted string. The variables must be passed as separate quoted arguments or the values must be sanitized before use in shell string interpolation.

Locations:

- `action.yml:450`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

1. Pinned `google-github-actions/auth@v3` to SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` and `actions/upload-artifact@v6` to SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f`.
2. Fixed github-env-injection in 'Sanitize workflow name' step by adding `SANITIZED=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` before writing to `$GITHUB_OUTPUT`, and also quoted `$GITHUB_OUTPUT`.
3. Fixed script-injection in 'Upload Telemetry to Google Cloud' step by replacing double-quoted `sed -e "s#...#${VAR}#g"` expressions (which allow bash command substitution) with `awk -v proj=... -v repo=... -v runid=...` variable assignments, which pass values as literal strings without shell expansion.

