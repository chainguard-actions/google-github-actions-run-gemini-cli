<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable full-length SHA commits, making them vulnerable to supply-chain attacks if the tag is moved:
- `google-github-actions/auth@v3` (tag, not a 40-char SHA) — marked `# ratchet:exclude` but still unpinned
- `actions/upload-artifact@v6` (tag, not a 40-char SHA) — marked `# ratchet:exclude` but still unpinned

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly SHA-pinned.

Locations:

- `action.yml:196`
- `action.yml:228`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced two mutable tag references with immutable full-length SHA commits:
- `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line 196)
- `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line 228)

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) was already correctly SHA-pinned and was left unchanged. The `# ratchet:exclude` comments were replaced with human-readable tag comments to preserve readability.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml. Added `SAFE_NAME=$(printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r')` to strip newline and carriage return characters from the workflow name before processing. Changed subsequent pipeline to use `printf '%s' "${SAFE_NAME}"` instead of `echo "${WORKFLOW_NAME}"` to avoid any echo-related issues. Also properly quoted `"$GITHUB_OUTPUT"` in the redirect. This prevents a crafted workflow name containing embedded newlines from injecting additional key=value pairs into $GITHUB_OUTPUT.

