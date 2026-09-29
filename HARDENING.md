<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved:
1. `uses: 'google-github-actions/auth@v3'` — tag `v3` is mutable (marked `# ratchet:exclude` but still unsafe)
2. `uses: 'actions/upload-artifact@v6'` — tag `v6` is mutable (marked `# ratchet:exclude` but still unsafe)
These should be pinned to their full SHA digests, e.g. `uses: 'google-github-actions/auth@<40-char-sha> # v3'`.

Locations:

- `action.yml:196`
- `action.yml:362`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned both mutable tag references to full 40-character SHA commit hashes:
1. `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line 196)
2. `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line 362)
SHAs were resolved via lookup_action_sha. The `# ratchet:exclude` comments were replaced with version tag comments for readability.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml (around line 190). Added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` to strip newline and carriage return characters from the sanitized workflow name before writing to $GITHUB_OUTPUT. Changed the echo statement to use `$safe` instead of `$SANITIZED`, and also properly quoted `"$GITHUB_OUTPUT"`. This prevents an attacker-controlled workflow name containing newline characters from injecting additional key=value pairs into $GITHUB_OUTPUT.

