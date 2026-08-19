<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple files reference GitHub Actions using mutable tags or branch names instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved.

action.yml:
  - uses: 'google-github-actions/auth@v3' (tag)
  - uses: 'actions/upload-artifact@v6' (tag)

.github/workflows/draft-release.yml:
  - uses: 'google-github-actions/.github/.github/workflows/draft-release.yml@v3' (tag)

.github/workflows/evals-nightly.yml:
  - uses: 'actions/checkout@v4' (tag, appears twice)
  - uses: 'actions/setup-node@v4' (tag, appears twice)
  - uses: 'actions/upload-artifact@v4' (tag)
  - uses: 'actions/download-artifact@v4' (tag)

.github/workflows/gemini-invoke.yml:
  - uses: 'actions/checkout@v4' (tag)
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/gemini-issue-fixer.yml:
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/gemini-plan-execute.yml:
  - uses: 'actions/checkout@v4' (tag)
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/gemini-review.yml:
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/gemini-scheduled-triage.yml:
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/gemini-triage.yml:
  - uses: 'google-github-actions/run-gemini-cli@main' (branch)

.github/workflows/release.yml:
  - uses: 'google-github-actions/.github/.github/workflows/release.yml@v3' (tag)

Locations:

- `action.yml:196`
- `action.yml:213`
- `.github/workflows/draft-release.yml:14`
- `.github/workflows/evals-nightly.yml:23`
- `.github/workflows/evals-nightly.yml:41`
- `.github/workflows/evals-nightly.yml:44`
- `.github/workflows/evals-nightly.yml:68`
- `.github/workflows/evals-nightly.yml:78`
- `.github/workflows/evals-nightly.yml:84`
- `.github/workflows/gemini-invoke.yml:28`
- `.github/workflows/gemini-invoke.yml:33`
- `.github/workflows/gemini-issue-fixer.yml:32`
- `.github/workflows/gemini-plan-execute.yml:29`
- `.github/workflows/gemini-plan-execute.yml:34`
- `.github/workflows/gemini-review.yml:29`
- `.github/workflows/gemini-scheduled-triage.yml:47`
- `.github/workflows/gemini-triage.yml:47`
- `.github/workflows/release.yml:9`

### script-injection (severity: high)

Sub-rule (a): The 'Run Evaluations' step in evals-nightly.yml directly interpolates GitHub Actions expressions into shell commands in the run: block. The expressions ${{ matrix.eval-file }} and ${{ matrix.model }} are embedded directly in shell command strings, bypassing any shell quoting protections. Before the shell executes the script, YAML template substitution replaces these expressions with their values — allowing an attacker who can influence the matrix values to inject arbitrary shell commands.

Offending lines:
  BASE_NAME=$(basename "${{ matrix.eval-file }}" .eval.ts)
  REPORT_FILE="eval-results-${{ matrix.model }}-${BASE_NAME}.json"
  npm run test:evals -- "${{ matrix.eval-file }}" --reporter=json --outputFile="$REPORT_FILE" || true

Fix: Move the matrix values into env: variables and reference them as quoted shell variables (e.g., "$EVAL_FILE") in the run: block.

Locations:

- `.github/workflows/evals-nightly.yml:66`
- `.github/workflows/evals-nightly.yml:67`
- `.github/workflows/evals-nightly.yml:70`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by pinning to full 40-character SHA commit hashes: google-github-actions/auth@v3→7c6bc770..., actions/upload-artifact@v6→b7c566a7..., actions/upload-artifact@v4→ea165f8d..., actions/download-artifact@v4→d3f86a10..., actions/checkout@v4→11d5960a..., actions/setup-node@v4→49933ea5..., google-github-actions/run-gemini-cli@main→f5a57753..., google-github-actions/.github@v3→29c6d38e.... Fixed script injection in evals-nightly.yml 'Run Evaluations' step by moving matrix.eval-file and matrix.model expressions into env: block as EVAL_FILE and GEMINI_MODEL, then referencing them as plain shell variables in the run: block.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Sanitize workflow name' step in hardened/action/action.yml (around line 196). Added `safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')` immediately before the write to $GITHUB_OUTPUT, and changed the write to use `$safe` instead of `$SANITIZED`. Also properly quoted `"$GITHUB_OUTPUT"` in the redirect. This ensures any newline or carriage-return characters embedded in the untrusted `inputs.workflow_name` value are stripped before being written to the output, preventing header injection attacks.

