<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or hijacked. The `# ratchet:exclude` comments are tool annotations and do not constitute a security exemption.

1. `uses: 'google-github-actions/auth@v3'` — `@v3` is a mutable tag.
2. `uses: 'actions/upload-artifact@v6'` — `@v6` is a mutable tag.

The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly pinned to a full SHA and passes.

Locations:

- `action.yml:190`
- `action.yml:330`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned two mutable tag references to full commit SHAs: (1) google-github-actions/auth@v3 → @7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3; (2) actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6. The pnpm/action-setup reference was already correctly pinned and left unchanged.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml (around line 178). Added the required sanitization step: `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` immediately before writing to $GITHUB_OUTPUT, and changed the write to use `$safe` instead of `$SANITIZED`. Also quoted `$GITHUB_OUTPUT` for correctness. The sed pipeline already strips most special characters, but the explicit `tr -d '\n\r'` step is now present as required by the prescribed pattern.

