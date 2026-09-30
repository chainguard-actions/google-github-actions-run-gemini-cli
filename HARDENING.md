<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `google-github-actions/auth@v3` (line ~246, marked `# ratchet:exclude`)
- `actions/upload-artifact@v6` (line ~340, marked `# ratchet:exclude`)
These should be pinned to full SHA digests, e.g. `google-github-actions/auth@<40-char-sha> # v3`.

Locations:

- `action.yml:246`
- `action.yml:340`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned two mutable tag references in hardened/action/action.yml:
- `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3`
- `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6`
SHAs were resolved via lookup_action_sha against the live GitHub API.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in action.yml to use the required `printf '%s' ... | tr -d '\n\r'` sanitization pattern. The fix: (1) strips newlines/carriage returns from WORKFLOW_NAME at the start of the pipeline using `printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r'`, (2) applies the existing character sanitization pipeline (sed, xargs, tr), (3) applies a final `safe=$(printf '%s' "${SANITIZED}" | tr -d '\n\r')` to ensure the output value is clean before writing to GITHUB_OUTPUT, and (4) also fixed the unquoted `$GITHUB_OUTPUT` to `"$GITHUB_OUTPUT"` as a best practice.

