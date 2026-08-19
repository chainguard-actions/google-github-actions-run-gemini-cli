<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.19** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Sanitize workflow name' step writes the value of `inputs.workflow_name` (an untrusted input, passed via the `WORKFLOW_NAME` env var) to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization pattern immediately before the write. The sed-based sanitization (`sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]'`) is not the required pattern. The write is: `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:163`

### unpinned-uses (severity: high)

action.yml contains two unpinned `uses:` references: `google-github-actions/auth@v3` (tag, not a full SHA) and `actions/upload-artifact@v6` (tag, not a full SHA). These should be pinned to full 40-character commit SHAs to prevent supply-chain attacks.

Locations:

- `action.yml:178`
- `action.yml:330`

### unpinned-uses (severity: high)

Multiple workflow files reference `google-github-actions/run-gemini-cli@main` (a mutable branch ref, not a SHA) and external reusable workflows at `@v3` (a mutable tag). Failing references: `google-github-actions/run-gemini-cli@main` in gemini-invoke.yml, gemini-issue-fixer.yml, gemini-review.yml, gemini-scheduled-triage.yml, gemini-triage.yml; `google-github-actions/.github/.github/workflows/draft-release.yml@v3` in draft-release.yml; `google-github-actions/.github/.github/workflows/release.yml@v3` in release.yml.

Locations:

- `.github/workflows/gemini-invoke.yml:36`
- `.github/workflows/gemini-issue-fixer.yml:36`
- `.github/workflows/gemini-review.yml:36`
- `.github/workflows/gemini-scheduled-triage.yml:72`
- `.github/workflows/gemini-triage.yml:55`
- `.github/workflows/draft-release.yml:17`
- `.github/workflows/release.yml:8`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses

**Notes:**

Fixed 3 findings:
1. github-env-injection in action.yml 'Sanitize workflow name' step: added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
2. Pinned google-github-actions/auth@v3 → @7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 and actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f in action.yml.
3. Pinned google-github-actions/run-gemini-cli@main → @f5a57753971eb5f2734c70df7e796f2fcfbef6e7 in all 5 workflow files; pinned google-github-actions/.github reusable workflows @v3 → @29c6d38eeb974133b4b66401985f7c70cf4a6681 in draft-release.yml and release.yml.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/gemini-scheduled-triage.yml at line 68. The ISSUES variable (populated from `gh issue list --json number,title,body`, which includes user-controlled issue body content) was being written directly to $GITHUB_OUTPUT without newline sanitization. Fixed by introducing a SAFE_ISSUES variable that strips newlines and carriage returns using `printf '%s' "${ISSUES}" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. The raw ISSUES variable is still used for the jq length count, which is safe as it doesn't write to GITHUB_OUTPUT.

