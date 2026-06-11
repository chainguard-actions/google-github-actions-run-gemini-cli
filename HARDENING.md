<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tags instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (tag `v3`, line ~196)
- `actions/upload-artifact@v6` (tag `v6`, line ~355)
Both are marked `# ratchet:exclude` but still represent unpinned references. The `pnpm/action-setup` reference is correctly pinned to a full SHA and passes.

Locations:

- `action.yml:196`
- `action.yml:355`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from the untrusted input `inputs.workflow_name` (via the `WORKFLOW_NAME` env var) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). The pipeline used (`sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`) strips some special characters but does not apply the prescribed newline-stripping sanitization before the write:
```bash
SANITIZED=$(echo "${WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
```
A caller controlling `inputs.workflow_name` could potentially inject newlines into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three changes made to action.yml:
1. Pinned `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line ~196).
2. Pinned `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line ~355).
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and changed the echo to write `$safe` instead of `$SANITIZED`, and quoted `$GITHUB_OUTPUT`.

