<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `uses: 'google-github-actions/auth@v3'` (tagged, not SHA-pinned)
- `uses: 'actions/upload-artifact@v6'` (tagged, not SHA-pinned)

Note: `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly pinned to a SHA.

Locations:

- `action.yml:238`
- `action.yml:330`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from the untrusted input `inputs.workflow_name` (via the `WORKFLOW_NAME` env var) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The custom pipeline used (`sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`) does not satisfy the check's required sanitization pattern. A caller-controlled value containing newlines or special sequences could inject additional key=value pairs into GITHUB_OUTPUT.

Offending line: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`

The `WORKFLOW_NAME` env var is set to `${{ inputs.workflow_name }}`, which is fully attacker-controlled.

Locations:

- `action.yml:178`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Pinned `google-github-actions/auth@v3` to SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` with `# v3` comment.
2. Pinned `actions/upload-artifact@v6` to SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` with `# v6` comment.
3. Fixed the 'Sanitize workflow name' step: added `SAFE_WORKFLOW_NAME=$(printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r')` before the existing sanitization pipeline to strip newlines, preventing GITHUB_OUTPUT injection. Also quoted `"$GITHUB_OUTPUT"` in the echo redirect.

