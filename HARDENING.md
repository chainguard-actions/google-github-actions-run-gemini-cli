<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable tag refs instead of full 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `google-github-actions/auth@v3` (tag ref, not a SHA)
- `actions/upload-artifact@v6` (tag ref, not a SHA)

Both are marked `# ratchet:exclude` but remain unpinned. The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly SHA-pinned.

Locations:

- `action.yml:200`
- `action.yml:340`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (an untrusted caller-controlled input) to `$GITHUB_OUTPUT` without the required sanitization pipeline (`printf '%s' "$VAR" | tr -d '\n\r'`). The value flows through env var `WORKFLOW_NAME: '${{ inputs.workflow_name }}'` → `SANITIZED=$(echo "${WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')` → `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. While the `sed`+`xargs` pipeline strips most special characters, it does not apply the required `printf '%s' | tr -d '\n\r'` sanitization immediately before the write to `$GITHUB_OUTPUT`, leaving a potential newline-injection path into the output file.

Locations:

- `action.yml:174`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Three changes made to hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` → `@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 # v3`
2. Pinned `actions/upload-artifact@v6` → `@b7c566a772e6b6bfb58ed0dc250532a479d7789f # v6`
3. Fixed github-env-injection in the 'Sanitize workflow name' step: added `SANITIZED=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` immediately before the `echo "gh_workflow_name=$SANITIZED" >> "$GITHUB_OUTPUT"` write, and also quoted `$GITHUB_OUTPUT` for correctness.

