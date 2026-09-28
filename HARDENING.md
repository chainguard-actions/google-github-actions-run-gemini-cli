<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
1. `uses: 'google-github-actions/auth@v3'` — tag `v3` is mutable (marked `# ratchet:exclude` but still unpinned)
2. `uses: 'actions/upload-artifact@v6'` — tag `v6` is mutable (marked `# ratchet:exclude` but still unpinned)
These should be pinned to their full commit SHAs (e.g. `google-github-actions/auth@<40-char-sha>`).
Note: `pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061` is correctly pinned.

Locations:

- `action.yml:213`
- `action.yml:370`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from the untrusted input `inputs.workflow_name` (via the `WORKFLOW_NAME` env var) to `$GITHUB_OUTPUT` without the required sanitization pattern (`printf '%s' "$VAR" | tr -d '\n\r'`) applied immediately before the write. While the `sed 's/[^ a-zA-Z0-9-]//g'` pipeline does strip most special characters, the mandated sanitization step is absent. A calling workflow could supply a crafted `workflow_name` value containing newlines that survive the sed filter (e.g. via multi-line default expressions), potentially injecting additional key=value pairs into `$GITHUB_OUTPUT`.

Offending line:
```
echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
```
where `SANITIZED` is derived from `WORKFLOW_NAME: '${{ inputs.workflow_name }}'`.

Locations:

- `action.yml:196`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Pinned google-github-actions/auth@v3 to full SHA 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 (keeping # v3 comment). 2. Pinned actions/upload-artifact@v6 to full SHA b7c566a772e6b6bfb58ed0dc250532a479d7789f (keeping # v6 comment). 3. Fixed github-env-injection in 'Sanitize workflow name' step: added tr -d '\n\r' at the start of the pipeline to strip newlines from WORKFLOW_NAME before sed processing, and added the mandatory `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` sanitization immediately before writing to $GITHUB_OUTPUT. Also quoted $GITHUB_OUTPUT reference.

