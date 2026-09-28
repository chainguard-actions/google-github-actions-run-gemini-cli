<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved:
- `google-github-actions/auth@v2` (line ~185) — tagged ref, not SHA-pinned
- `actions/upload-artifact@v4` (line ~340) — tagged ref, not SHA-pinned

Note: `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly SHA-pinned.

Locations:

- `action.yml:185`
- `action.yml:340`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (an untrusted input) to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization pattern. The value flows through `WORKFLOW_NAME` env var → `SANITIZED` shell variable → `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. While the `sed 's/[^ a-zA-Z0-9-]//g'` pipeline strips most special characters, the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`) is not applied immediately before the write to `$GITHUB_OUTPUT`.

Locations:

- `action.yml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three fixes applied to hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v2` to `google-github-actions/auth@c200f3691d83b41bf9bbd8638997a462592937ed # v2`
2. Pinned `actions/upload-artifact@v4` to `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `SANITIZED=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` immediately before writing to `$GITHUB_OUTPUT`, and quoted `$GITHUB_OUTPUT` in the echo command.

