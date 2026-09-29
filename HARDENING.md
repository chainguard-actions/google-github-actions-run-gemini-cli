<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: 'google-github-actions/auth@v3'` (tagged ref, not a SHA)
- `uses: 'actions/upload-artifact@v6'` (tagged ref, not a SHA)
Both are marked `# ratchet:exclude` but remain unpinned. The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly SHA-pinned.

Locations:

- `action.yml:240`
- `action.yml:460`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes `$SANITIZED` — a value derived from the untrusted input `inputs.workflow_name` (via the `WORKFLOW_NAME` env var) — to `$GITHUB_OUTPUT` without the required sanitization pipeline (`printf '%s' ... | tr -d '\n\r'`) immediately before the write. The value passes through `sed | xargs | tr` but this does not satisfy the required sanitization pattern. An attacker-controlled `workflow_name` input containing newline sequences could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

Offending line: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:222`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` to SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` (# v3)
2. Pinned `actions/upload-artifact@v6` to SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` (# v6)
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and write `$safe` (not `$SANITIZED`) to `$GITHUB_OUTPUT`, preventing newline injection via the `workflow_name` input.

