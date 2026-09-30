<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Both are marked `# ratchet:exclude` but that does not make them safe:
- `google-github-actions/auth@v3` — mutable tag
- `actions/upload-artifact@v6` — mutable tag

Locations:

- `action.yml:196`
- `action.yml:311`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step sets the `WORKFLOW_NAME` env var from `inputs.workflow_name` (attacker-controllable), processes it through `sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`, and writes the result to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. The `sed` command strips most special characters but does NOT strip embedded newlines (sed processes line-by-line). A newline injected via `inputs.workflow_name` could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent step outputs. The offending line is: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`

Locations:

- `action.yml:170`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` to full SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` (# v3).
2. Pinned `actions/upload-artifact@v6` to full SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` (# v6).
3. Fixed github-env-injection in 'Sanitize workflow name' step: added `SAFE=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` and write `$SAFE` (not `$SANITIZED`) to `$GITHUB_OUTPUT`, preventing newline injection that could overwrite subsequent step outputs.

