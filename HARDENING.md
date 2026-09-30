<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or overwritten. `google-github-actions/auth@v3` and `actions/upload-artifact@v6` are both tagged refs (marked `# ratchet:exclude`). The third reference (`pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061`) is correctly SHA-pinned.

Locations:

- `action.yml:214`
- `action.yml:390`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (an untrusted caller-controlled input) to `$GITHUB_OUTPUT` without the required sanitization pipeline (`printf '%s' ... | tr -d '\n\r'`). The value flows through `WORKFLOW_NAME` env var → `sed | xargs | tr` → `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. While `xargs` incidentally strips newlines, the prescribed sanitization step is absent. An attacker-controlled `workflow_name` input containing newline sequences could inject arbitrary key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:187`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three security issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v3` to full SHA `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` (line 214).
2. Pinned `actions/upload-artifact@v6` to full SHA `b7c566a772e6b6bfb58ed0dc250532a479d7789f` (line 390).
3. Fixed github-env-injection in the 'Sanitize workflow name' step (line 187): added `printf '%s' ... | tr -d '\n\r'` sanitization at both the input stage (stripping newlines from WORKFLOW_NAME before sed processing) and before writing to $GITHUB_OUTPUT, preventing an attacker-controlled `workflow_name` input from injecting arbitrary key=value pairs into $GITHUB_OUTPUT via embedded newlines.

