<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: 'google-github-actions/auth@v2'` — mutable tag `v2` (marked `# ratchet:exclude` but still unpinned)
- `uses: 'actions/upload-artifact@v4'` — mutable tag `v4` (marked `# ratchet:exclude` but still unpinned)
The third reference `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly pinned to a SHA.

Locations:

- `action.yml:200`
- `action.yml:355`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (an attacker-controllable input) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) immediately before the write. The env var `WORKFLOW_NAME` is set from `${{ inputs.workflow_name }}` and then processed through `sed/xargs/tr` into `$SANITIZED`, but the final write `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT` does not apply the mandatory newline-stripping sanitization pipeline before writing to the special environment file. A crafted input containing embedded newlines could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:176`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v2` to immutable SHA `c200f3691d83b41bf9bbd8638997a462592937ed` (# v2)
2. Pinned `actions/upload-artifact@v4` to immutable SHA `ea165f8d65b6e75b540449e92b4886f43607fa02` (# v4)
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` before writing to GITHUB_OUTPUT, and updated the write to use `$safe` with a quoted `"$GITHUB_OUTPUT"` reference.

