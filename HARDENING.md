<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.18

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.18** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks:

**action.yml:**
- `google-github-actions/auth@v2` (ratchet:exclude)
- `actions/upload-artifact@v4` (ratchet:exclude)

**Workflow files:**
- `google-github-actions/run-gemini-cli@main` (in gemini-invoke.yml, gemini-issue-fixer.yml, gemini-review.yml, gemini-scheduled-triage.yml, gemini-triage.yml)
- `google-github-actions/.github/.github/workflows/draft-release.yml@v3` (in draft-release.yml)
- `google-github-actions/.github/.github/workflows/release.yml@v3` (in release.yml)

Locations:

- `action.yml:205`
- `action.yml:380`
- `.github/workflows/gemini-invoke.yml:33`
- `.github/workflows/gemini-issue-fixer.yml:38`
- `.github/workflows/gemini-review.yml:35`
- `.github/workflows/gemini-scheduled-triage.yml:75`
- `.github/workflows/gemini-triage.yml:55`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/release.yml:8`

### github-env-injection (severity: high)

Two steps write untrusted/user-controlled values to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization:

1. **action.yml `sanitize_workflow_name` step**: The env var `WORKFLOW_NAME` is set from `inputs.workflow_name` (caller-controlled). The value is processed through `sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'` and written as `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. The required `printf '%s' ... | tr -d '\n\r'` sanitization pattern is not applied immediately before the write.

2. **gemini-scheduled-triage.yml `find_issues` step**: The variable `ISSUES` is populated from `gh issue list --json number,title,body`, which includes user-controlled issue titles and bodies that can contain arbitrary newlines and special characters. It is written directly to `$GITHUB_OUTPUT` as `echo "issues_to_triage=${ISSUES}" >> "${GITHUB_OUTPUT}"` with no sanitization whatsoever, allowing newline injection to poison subsequent output parsing.

Locations:

- `action.yml:185`
- `.github/workflows/gemini-scheduled-triage.yml:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Fixed all unpinned `uses:` references by resolving them to full 40-character commit SHAs using lookup_action_sha: (1) google-github-actions/auth@v2 → @c200f3691d83b41bf9bbd8638997a462592937ed in action.yml; (2) actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 in action.yml; (3) google-github-actions/run-gemini-cli@main → @f5a57753971eb5f2734c70df7e796f2fcfbef6e7 in all 5 workflow files; (4) google-github-actions/.github workflows @v3 → @29c6d38eeb974133b4b66401985f7c70cf4a6681 in draft-release.yml and release.yml. Fixed github-env-injection in two locations: (1) action.yml sanitize_workflow_name step now uses `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` before writing to GITHUB_OUTPUT; (2) gemini-scheduled-triage.yml find_issues step now uses `safe_issues=$(printf '%s' "${ISSUES}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.

