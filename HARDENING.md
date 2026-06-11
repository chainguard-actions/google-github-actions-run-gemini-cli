<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable tag refs instead of immutable full 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved:
- `google-github-actions/auth@v3` (marked `# ratchet:exclude`)
- `actions/upload-artifact@v6` (marked `# ratchet:exclude`)

These should be pinned to full SHA digests, e.g. `google-github-actions/auth@<40-char-sha> # v3`.

Locations:

- `action.yml:247`
- `action.yml:338`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned two mutable tag references to immutable full SHA digests:
1. `google-github-actions/auth@v3` → `google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3` (line 247)
2. `actions/upload-artifact@v6` → `actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6` (line 338)

The `# ratchet:exclude` comments were replaced with human-readable version tag comments. SHAs were resolved via lookup_action_sha.

