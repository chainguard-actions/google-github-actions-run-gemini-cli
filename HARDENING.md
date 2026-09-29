<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (line ~190, marked `# ratchet:exclude` but still unpinned)
- `actions/upload-artifact@v6` (line ~310, marked `# ratchet:exclude` but still unpinned)

Note: `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly pinned to a full SHA.

Locations:

- `action.yml:190`
- `action.yml:310`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes `$SANITIZED` — derived from the `$WORKFLOW_NAME` env var, which is set from `inputs.workflow_name` (an attacker-controllable value) — to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization:

```yaml
run: |
  SANITIZED=$(echo "${WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
  echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
env:
  WORKFLOW_NAME: '${{ inputs.workflow_name }}'
```

The `sed` command processes input line-by-line and does NOT strip the newline characters that separate lines — only non-matching characters within each line are removed. A `WORKFLOW_NAME` value containing an embedded newline (e.g. `foo\nbar=injected`) will produce a multi-line `$SANITIZED`, and the `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT` write will inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other step outputs. The fix is to apply `printf '%s' "$SANITIZED" | tr -d '\n\r'` immediately before the write.

Locations:

- `action.yml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three fixes applied to hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` to full SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` (# v3 comment preserved).
2. Pinned `actions/upload-artifact@v6` to full SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` (# v6 comment preserved).
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `SANITIZED=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` to strip embedded newlines before writing to GITHUB_OUTPUT, and quoted `"$GITHUB_OUTPUT"` in the redirect.

