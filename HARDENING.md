<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tag refs instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (tag `v3`)
- `actions/upload-artifact@v6` (tag `v6`)

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly pinned to a SHA and passes. Both failing references carry `# ratchet:exclude` comments but are still unpinned.

Locations:

- `action.yml:196`
- `action.yml:345`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes `$SANITIZED` — derived from the untrusted `inputs.workflow_name` input — to `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`) applied immediately before the write.

The sanitization pipeline used (`sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`) processes input line-by-line via `sed` and does not strip embedded newlines from the resulting value. A calling workflow could supply a crafted multi-line `workflow_name` value that injects additional `key=value` pairs into `$GITHUB_OUTPUT`, potentially overwriting outputs from other steps.

Offending line:
```bash
echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
```

Fix — apply `tr -d '\n\r'` before writing:
```bash
safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')
echo "gh_workflow_name=$safe" >> "$GITHUB_OUTPUT"
```

Locations:

- `action.yml:180`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` to full SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` (kept `# v3` comment for readability).
2. Pinned `actions/upload-artifact@v6` to full SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` (kept `# v6` comment for readability).
3. Fixed github-env-injection in the 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and changed the GITHUB_OUTPUT write to use `$safe` instead of `$SANITIZED`, preventing newline injection from the untrusted `inputs.workflow_name` value.

