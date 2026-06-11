<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml contains two `uses:` references pinned to mutable tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (tag reference, marked `# ratchet:exclude`)
- `actions/upload-artifact@v6` (tag reference, marked `# ratchet:exclude`)

The `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` reference is correctly pinned to a SHA and is not a finding.

Locations:

- `action.yml:196`
- `action.yml:231`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned two mutable tag references to full commit SHAs: (1) google-github-actions/auth@v3 → @7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3; (2) actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6. The ratchet:exclude comments were replaced with human-readable tag comments. The already-pinned pnpm/action-setup reference was left unchanged.

