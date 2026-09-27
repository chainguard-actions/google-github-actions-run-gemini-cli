<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable full SHA digests, making them vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised.

1. `uses: 'google-github-actions/auth@v3'` — uses mutable tag `v3` (marked `# ratchet:exclude` but still unpinned)
2. `uses: 'actions/upload-artifact@v6'` — uses mutable tag `v6` (marked `# ratchet:exclude` but still unpinned)

Both should be pinned to a full 40-character commit SHA, e.g. `uses: google-github-actions/auth@<40-char-sha> # v3`.

Locations:

- `action.yml:232`
- `action.yml:390`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned both mutable tag references to full commit SHAs:
1. `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line 232)
2. `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line 390)

Both were previously marked `# ratchet:exclude` but still used mutable tags. The `ratchet:exclude` comments have been replaced with the version tag comments for readability.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml (line ~174) by adding the required newline-stripping sanitization pattern. After the existing sed/xargs/tr sanitization produces SANITIZED, a new line `SAFE=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` strips any remaining newline/carriage-return characters before writing `gh_workflow_name=$SAFE` to $GITHUB_OUTPUT. Also properly quoted `"$GITHUB_OUTPUT"` in the echo statement.

