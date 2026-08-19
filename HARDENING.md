<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml and workflow files are pinned to mutable tags or branches instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references:
- action.yml: `google-github-actions/auth@v3` (tag, not SHA)
- action.yml: `actions/upload-artifact@v6` (tag, not SHA)
- .github/workflows/draft-release.yml: `google-github-actions/.github/.github/workflows/draft-release.yml@v3` (tag, not SHA)
- .github/workflows/release.yml: `google-github-actions/.github/.github/workflows/release.yml@v3` (tag, not SHA)
- .github/workflows/gemini-invoke.yml: `google-github-actions/run-gemini-cli@main` (branch, not SHA)
- .github/workflows/gemini-issue-fixer.yml: `google-github-actions/run-gemini-cli@main` (branch, not SHA)
- .github/workflows/gemini-review.yml: `google-github-actions/run-gemini-cli@main` (branch, not SHA)
- .github/workflows/gemini-scheduled-triage.yml: `google-github-actions/run-gemini-cli@main` (branch, not SHA)
- .github/workflows/gemini-triage.yml: `google-github-actions/run-gemini-cli@main` (branch, not SHA)
The `# ratchet:exclude` comments are tooling annotations and do not mitigate the supply-chain risk.

Locations:

- `action.yml:196`
- `action.yml:330`
- `.github/workflows/draft-release.yml:15`
- `.github/workflows/release.yml:9`
- `.github/workflows/gemini-invoke.yml:32`
- `.github/workflows/gemini-issue-fixer.yml:37`
- `.github/workflows/gemini-review.yml:34`
- `.github/workflows/gemini-scheduled-triage.yml:72`
- `.github/workflows/gemini-triage.yml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned all 9 unpinned `uses:` references to full 40-character commit SHAs:
- action.yml: `google-github-actions/auth@v3` → `@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` # v3
- action.yml: `actions/upload-artifact@v6` → `@b7c566a772e6b6bfb58ed0dc250532a479d7789f` # v6
- .github/workflows/draft-release.yml: `google-github-actions/.github/...@v3` → `@29c6d38eeb974133b4b66401985f7c70cf4a6681` # v3
- .github/workflows/release.yml: `google-github-actions/.github/...@v3` → `@29c6d38eeb974133b4b66401985f7c70cf4a6681` # v3
- .github/workflows/gemini-invoke.yml: `google-github-actions/run-gemini-cli@main` → `@f5a57753971eb5f2734c70df7e796f2fcfbef6e7` # main
- .github/workflows/gemini-issue-fixer.yml: `google-github-actions/run-gemini-cli@main` → `@f5a57753971eb5f2734c70df7e796f2fcfbef6e7` # main
- .github/workflows/gemini-review.yml: `google-github-actions/run-gemini-cli@main` → `@f5a57753971eb5f2734c70df7e796f2fcfbef6e7` # main
- .github/workflows/gemini-scheduled-triage.yml: `google-github-actions/run-gemini-cli@main` → `@f5a57753971eb5f2734c70df7e796f2fcfbef6e7` # main
- .github/workflows/gemini-triage.yml: `google-github-actions/run-gemini-cli@main` → `@f5a57753971eb5f2734c70df7e796f2fcfbef6e7` # main

A match in scripts/generate-examples.sh was inspected and found to be a string literal inside a sed substitution command, not an actual workflow `uses:` reference — no change needed there.

