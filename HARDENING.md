<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (line ~215) — tagged `# ratchet:exclude` but still unpinned
- `actions/upload-artifact@v6` (line ~370) — tagged `# ratchet:exclude` but still unpinned
These should be pinned to immutable commit SHAs.

Locations:

- `action.yml:215`
- `action.yml:370`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (an attacker-controlled input) to `$GITHUB_OUTPUT` without the required newline-stripping sanitization (`printf '%s' ... | tr -d '\n\r'`). The pipeline `sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'` strips special characters but does NOT strip newline characters. A crafted `workflow_name` input containing a newline could inject additional key=value pairs into GITHUB_OUTPUT. The offending line is: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three fixes applied to hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` → `@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3`
2. Pinned `actions/upload-artifact@v6` → `@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6`
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `tr -d '\n\r'` at the start of the pipeline (on the input) and again before writing to GITHUB_OUTPUT, and quoted `$GITHUB_OUTPUT` for correctness.

