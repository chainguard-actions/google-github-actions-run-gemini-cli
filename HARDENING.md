<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tags rather than full 40-character commit SHAs:
- `uses: 'google-github-actions/auth@v2'` (tag `v2`) — marked `# ratchet:exclude`, meaning it is intentionally excluded from SHA-pinning but is still a mutable reference vulnerable to supply-chain attacks.
- `uses: 'actions/upload-artifact@v4'` (tag `v4`) — also marked `# ratchet:exclude`.
The third reference `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly pinned to a SHA.

Locations:

- `action.yml:237`
- `action.yml:370`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (which defaults to `${{ github.workflow }}`, a caller-controlled value) to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The pipeline `sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'` does collapse whitespace via `xargs`, but the mandated `printf '%s' "$VAR" | tr -d '\n\r'` pattern is not applied immediately before the write. The offending line is:
```
echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
```
A caller supplying a crafted `workflow_name` input (or a workflow setting `github.workflow` to a value containing newlines) could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:207`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three changes made to action.yml:
1. Pinned `google-github-actions/auth@v2` → `google-github-actions/auth@c200f3691d83b41bf9bbd8638997a462592937ed # v2`
2. Pinned `actions/upload-artifact@v4` → `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and changed the GITHUB_OUTPUT write to use `$safe` instead of `$SANITIZED`, and quoted `"$GITHUB_OUTPUT"` for good measure.

