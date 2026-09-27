<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved:
- `google-github-actions/auth@v2` (tag `v2`, marked `# ratchet:exclude`)
- `actions/upload-artifact@v4` (tag `v4`, marked `# ratchet:exclude`)

These should be pinned to full SHA digests (e.g. `google-github-actions/auth@<40-hex-sha>`).

Locations:

- `action.yml:200`
- `action.yml:370`

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes a value derived from `inputs.workflow_name` (via the `WORKFLOW_NAME` env var) to `$GITHUB_OUTPUT` without the required `printf '%s' "$VAR" | tr -d '\n\r'` sanitization. The pipeline used (`sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`) strips many special characters but does not apply the mandatory newline-stripping sanitization step before the write:

```bash
SANITIZED=$(echo "${WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
```

A calling workflow can supply a crafted `workflow_name` input (whose default is `${{ github.workflow }}`) that injects additional key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `action.yml:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed three issues in hardened/action/action.yml:
1. Pinned `google-github-actions/auth@v2` to full SHA `c200f3691d83b41bf9bbd8638997a462592937ed # v2`
2. Pinned `actions/upload-artifact@v4` to full SHA `ea165f8d65b6e75b540449e92b4886f43607fa02 # v4`
3. Fixed github-env-injection in 'Sanitize workflow name' step: replaced `echo "${WORKFLOW_NAME}"` with `printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r'` to strip newlines before processing, and added a second `printf '%s' "$SANITIZED" | tr -d '\n\r'` sanitization step before writing to `$GITHUB_OUTPUT`. Also quoted `$GITHUB_OUTPUT` for correctness.

