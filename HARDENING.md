<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `uses: 'google-github-actions/auth@v3'` — mutable tag `v3` (marked `# ratchet:exclude` but still unpinned)
- `uses: 'actions/upload-artifact@v6'` — mutable tag `v6` (marked `# ratchet:exclude` but still unpinned)

These should be pinned to their full SHA digests, e.g. `google-github-actions/auth@<40-char-sha> # v3`.

Locations:

- `action.yml:161`
- `action.yml:290`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned both mutable tag references in hardened/action/action.yml to full 40-character SHA digests: `google-github-actions/auth@v3` → `@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line 161), and `actions/upload-artifact@v6` → `@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line 290). The `# ratchet:exclude` comments were replaced with the tag name as a readability comment.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml (around line 184). Added the mandatory `printf '%s' ... | tr -d '\n\r'` sanitization pattern in two places: (1) at the start of the pipeline replacing `echo` with `printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r'` to strip newlines before sed processing, and (2) immediately before writing to $GITHUB_OUTPUT with `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')`. Also quoted `$GITHUB_OUTPUT` as best practice. This prevents a crafted workflow_name containing newlines from injecting additional key=value pairs into $GITHUB_OUTPUT.

