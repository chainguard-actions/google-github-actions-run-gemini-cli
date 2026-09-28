<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Both are annotated with `# ratchet:exclude` (a tool hint), but that annotation provides no security guarantee.

- `uses: 'google-github-actions/auth@v3'` — `@v3` is a mutable tag
- `uses: 'actions/upload-artifact@v6'` — `@v6` is a mutable tag

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly pinned to a full SHA and passes.

Locations:

- `action.yml:237`
- `action.yml:290`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned two mutable tag references to full commit SHAs:
- `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` # v3
- `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f` # v6

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) was already pinned and required no changes.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in hardened/action/action.yml (around line 196). Added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` after computing SANITIZED, and changed the echo to write `$safe` instead of `$SANITIZED` to $GITHUB_OUTPUT. Also quoted `"$GITHUB_OUTPUT"` for correctness. This prevents a multi-line `workflow_name` input from injecting additional key=value pairs into GITHUB_OUTPUT.

