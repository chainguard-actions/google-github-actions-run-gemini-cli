<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references two actions using mutable version tags instead of full SHA digests: 'google-github-actions/auth@v3' and 'actions/upload-artifact@v6'. These are marked ratchet:exclude but still represent unpinned supply-chain references.

Locations:

- `action.yml:220`
- `action.yml:390`

### unpinned-uses (severity: high)

evals-nightly.yml uses three unpinned action references: 'actions/checkout@v4', 'actions/setup-node@v4', and 'actions/upload-artifact@v4'. All use mutable version tags instead of full 40-character SHA digests.

Locations:

- `.github/workflows/evals-nightly.yml:30`
- `.github/workflows/evals-nightly.yml:34`
- `.github/workflows/evals-nightly.yml:52`

### unpinned-uses (severity: high)

draft-release.yml uses an unpinned reusable workflow reference: 'google-github-actions/.github/.github/workflows/draft-release.yml@v3'. This uses a mutable version tag instead of a full SHA digest.

Locations:

- `.github/workflows/draft-release.yml:16`

### unpinned-uses (severity: high)

release.yml uses an unpinned reusable workflow reference: 'google-github-actions/.github/.github/workflows/release.yml@v3'. This uses a mutable version tag instead of a full SHA digest.

Locations:

- `.github/workflows/release.yml:10`

### unpinned-uses (severity: high)

gemini-invoke.yml uses two unpinned action references: 'actions/checkout@v4' and 'google-github-actions/run-gemini-cli@main'. The @main branch reference is especially risky as it tracks a mutable branch head.

Locations:

- `.github/workflows/gemini-invoke.yml:33`
- `.github/workflows/gemini-invoke.yml:37`

### unpinned-uses (severity: high)

gemini-issue-fixer.yml uses an unpinned action reference: 'google-github-actions/run-gemini-cli@main'. The @main branch reference tracks a mutable branch head and is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/gemini-issue-fixer.yml:40`

### unpinned-uses (severity: high)

gemini-plan-execute.yml uses two unpinned action references: 'actions/checkout@v4' and 'google-github-actions/run-gemini-cli@main'. The @main branch reference tracks a mutable branch head.

Locations:

- `.github/workflows/gemini-plan-execute.yml:34`
- `.github/workflows/gemini-plan-execute.yml:38`

### unpinned-uses (severity: high)

gemini-review.yml uses an unpinned action reference: 'google-github-actions/run-gemini-cli@main'. The @main branch reference tracks a mutable branch head and is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/gemini-review.yml:35`

### unpinned-uses (severity: high)

gemini-scheduled-triage.yml uses an unpinned action reference: 'google-github-actions/run-gemini-cli@main'. The @main branch reference tracks a mutable branch head and is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/gemini-scheduled-triage.yml:64`

### unpinned-uses (severity: high)

gemini-triage.yml uses an unpinned action reference: 'google-github-actions/run-gemini-cli@main'. The @main branch reference tracks a mutable branch head and is vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/gemini-triage.yml:51`

### script-injection (severity: high)

evals-nightly.yml has two run: blocks that directly interpolate ${{ matrix.model }} into shell command strings (sub-rule a). Although matrix values are defined in the workflow itself, any ${{ ... }} expression inside a run: block is a script-injection finding. Offending lines: (1) `npm run test:evals -- --reporter=json --outputFile=eval-results-${{ matrix.model }}.json` and (2) `npx tsx scripts/aggregate_evals.ts "eval-results-${{ matrix.model }}.json" >> "$GITHUB_STEP_SUMMARY"`. The fix is to use the $GEMINI_MODEL env var (already set in the env: block) instead of the expression.

Locations:

- `.github/workflows/evals-nightly.yml:47`
- `.github/workflows/evals-nightly.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by pinning to full 40-character SHA digests:
- action.yml: google-github-actions/auth@v3 → @7c6bc770dae815cd3e89ee6cdf493a5fab2cc093, actions/upload-artifact@v6 → @b7c566a772e6b6bfb58ed0dc250532a479d7789f
- evals-nightly.yml: actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5, actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020, actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
- draft-release.yml and release.yml: google-github-actions/.github reusable workflow @v3 → @29c6d38eeb974133b4b66401985f7c70cf4a6681
- gemini-invoke.yml, gemini-plan-execute.yml: actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
- gemini-invoke.yml, gemini-issue-fixer.yml, gemini-plan-execute.yml, gemini-review.yml, gemini-scheduled-triage.yml, gemini-triage.yml: google-github-actions/run-gemini-cli@main → @f5a57753971eb5f2734c70df7e796f2fcfbef6e7

Fixed script injection in evals-nightly.yml: replaced ${{ matrix.model }} in two run: blocks with the $GEMINI_MODEL environment variable (already set in the env: block for the 'Run Evaluations' step; added env: block to 'Job Summary' step). The ${{ matrix.model }} expressions remaining in YAML fields (name:, with: path:, env:) are not shell injection risks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Find untriaged issues' step in .github/workflows/gemini-scheduled-triage.yml. The raw JSON output from `gh issue list` (which can contain newlines in issue bodies) is now sanitized before being written to $GITHUB_OUTPUT. Added `SAFE_ISSUES="$(printf '%s' "${ISSUES}" | tr -d '\n\r')"` and changed the echo to use `${SAFE_ISSUES}` instead of `${ISSUES}`, preventing newline injection attacks via malicious issue bodies.

