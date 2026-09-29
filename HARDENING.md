<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or compromised. The `# ratchet:exclude` comments are tool annotations that do not provide any security guarantee:
- `uses: 'google-github-actions/auth@v2'` — mutable tag ref
- `uses: 'actions/upload-artifact@v4'` — mutable tag ref

These should be pinned to their full SHA digests, e.g. `uses: 'google-github-actions/auth@<40-char-sha>' # v2`.

Locations:

- `action.yml:200`
- `action.yml:330`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned both mutable tag references in hardened/action/action.yml to full commit SHAs:
- `google-github-actions/auth@v2` → `google-github-actions/auth@c200f3691d83b41bf9bbd8638997a462592937ed # v2` (line 200)
- `actions/upload-artifact@v4` → `actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 # v4` (line 330)
The `# ratchet:exclude` annotations were replaced with the tag name as a comment for readability.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml (around line 189). Added `SAFE=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and changed the echo to write `$SAFE` instead of `$SANITIZED` to $GITHUB_OUTPUT. Also quoted `$GITHUB_OUTPUT` for best practice. The existing sed/xargs/tr pipeline strips most special characters but does not satisfy the required newline-stripping pattern; the new line ensures newlines and carriage returns are removed before the value is written to $GITHUB_OUTPUT, preventing header injection attacks.

