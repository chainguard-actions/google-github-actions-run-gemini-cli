<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.21

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.21** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In step 'Run Gemini CLI', the multi-line outputs use a fixed heredoc delimiter: `echo "gemini_response<<EOF" >> "${GITHUB_OUTPUT}"` followed by `echo "${RESPONSE}" >> "${GITHUB_OUTPUT}"` (or `cat "${TEMP_STDOUT}"`), then `echo "EOF"`. gemini_errors is written the same way from ERROR_JSON / TEMP_STDERR. RESPONSE and the stdout/stderr content come from a model run on inputs.prompt, which can be attacker-influenced. A line equal to 'EOF' in that content ends the block early, and every later line becomes a new output entry. Fix: build the delimiter from a secure random source, e.g. `EOF="EOF_$(od -An -N16 -tx1 /dev/urandom | tr -d ' \n')"`.

Locations:

- `action.yml:341`
- `action.yml:349`

### github-env-injection (severity: high)

In step 'Sanitize workflow name', a value built from inputs.workflow_name (via env WORKFLOW_NAME) is written with `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. The required pipeline `printf '%s' ... | tr -d '\n\r'` is not applied right before the write. The sed/xargs filtering probably removes newlines anyway, but it is not the required step.

Locations:

- `action.yml:186`

### unpinned-uses (severity: high)

action.yml has composite-step `uses:` references pinned to tags instead of 40-character commit SHAs: `uses: 'google-github-actions/auth@v3' # ratchet:exclude` and `uses: 'actions/upload-artifact@v6' # ratchet:exclude`. (pnpm/action-setup is correctly pinned to a SHA.)

Locations:

- `action.yml:213`
- `action.yml:420`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 70ef1b1..d9937f0 100644
--- a/action.yml
+++ b/action.yml
@@ -16,7 +16,6 @@ name: 'Run Gemini CLI'
 author: 'Google LLC'
 description: |-
   Invoke the Gemini CLI from a GitHub Action.
-
 inputs:
   gcp_location:
     description: 'The Google Cloud location.'
@@ -91,7 +90,6 @@ inputs:
     description: 'The GitHub workflow name, used for telemetry purposes.'
     required: false
     default: '${{ github.workflow }}'
-
 outputs:
   summary:
     description: 'The summarized output from the Gemini CLI execution.'
@@ -99,10 +97,26 @@ outputs:
   error:
     description: 'The error output from the Gemini CLI execution, if any.'
     value: '${{ steps.gemini_run.outputs.gemini_errors }}'
-
 runs:
   using: 'composite'
   steps:
     - name: 'Validate Inputs'
       id: 'validate_inputs'
       shell: 'bash'
@@ -177,16 +191,15 @@ runs:
         INPUT_GCP_TOKEN_FORMAT: '${{ inputs.gcp_token_format }}'
         INPUT_USE_VERTEX_AI: '${{ inputs.use_vertex_ai }}'
         INPUT_USE_GEMINI_CODE_ASSIST: '${{ inputs.use_gemini_code_assist }}'
-
     - name: 'Sanitize workflow name'
       id: 'sanitize_workflow_name'
       shell: 'bash'
       run: |
-        SANITIZED=$(echo "${WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
-        echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT
+        SANITIZED=$(printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r' | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
+        safe=$(printf '%s' "$SANITIZED" | tr -d '\n\r')
+        echo "gh_workflow_name=$safe" >> "$GITHUB_OUTPUT"
       env:
         WORKFLOW_NAME: '${{ inputs.workflow_name }}'
-
     - name: 'Configure Gemini CLI'
       if: |-
         ${{ inputs.settings != '' }}
@@ -196,7 +209,6 @@ runs:
       shell: 'bash'
       env:
         SETTINGS: '${{ inputs.settings }}'
-
     - name: 'Install Custom Commands'
       shell: 'bash'
       run: |-
@@ -205,26 +217,23 @@ runs:
         cp -r "${GITHUB_ACTION_PATH}/.github/commands/"* .gemini/commands/
       env:
         GITHUB_ACTION_PATH: '${{ github.action_path }}'
-
     - name: 'Authenticate to Google Cloud'
       if: |-
         ${{ inputs.gcp_workload_identity_provider != '' }}
       id: 'auth'
       uses: 'google-github-actions/auth@v3' # ratchet:exclude
       with:
         project_id: '${{ inputs.gcp_project_id }}'
         workload_identity_provider: '${{ inputs.gcp_workload_identity_provider }}'
         service_account: '${{ inputs.gcp_service_account }}'
         token_format: '${{ inputs.gcp_token_format }}'
         access_token_scopes: '${{ inputs.gcp_access_token_scopes }}'
-
     - name: 'Install pnpm'
       if: |-
         ${{ inputs.use_pnpm == 'true' }}
       uses: 'pnpm/action-setup@41ff72655975bd51cab0327fa583b6e92b6d3061' # ratchet:pnpm/action-setup@v4
       with:
         version: 10
-
     - name: 'Install Gemini CLI'
       id: 'install'
       env:
@@ -270,7 +279,6 @@ runs:
             fi
           done
         fi
-
     - name: 'Run Gemini CLI'
       id: 'gemini_run'
       shell: 'bash'
@@ -414,15 +422,13 @@ runs:
         PROMPT: '${{ inputs.prompt }}'
         GEMINI_MODEL: '${{ inputs.gemini_model }}'
         GH_WORKFLOW_NAME: '${{ steps.sanitize_workflow_name.outputs.gh_workflow_name }}'
-
     - name: 'Upload Gemini CLI outputs'
       if: |-
         ${{ inputs.upload_artifacts == 'true' }}
       uses: 'actions/upload-artifact@v6' # ratchet:exclude
       with:
         name: 'gemini-output'
         path: 'gemini-artifacts/'
-
     - name: 'Upload Telemetry to Google Cloud'
       if: |-
         ${{ inputs.gcp_workload_identity_provider != '' }}
@@ -490,7 +496,6 @@ runs:
         GITHUB_ACTION_PATH: '${{ github.action_path }}'
         GITHUB_REPOSITORY: '${{ github.repository }}'
         GITHUB_RUN_ID: '${{ github.run_id }}'
-
 branding:
   icon: 'terminal'
   color: 'blue'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, prior-fix

**Notes:**

I fixed all three findings in action.yml. Nothing was run or tested after the edits.

1. **Fixed heredoc delimiter in 'Run Gemini CLI':** the `gemini_response` and `gemini_errors` outputs now each use a delimiter generated fresh on every run with `od -An -N16 -tx1 /dev/urandom`, instead of the fixed `EOF`. The output values themselves are written unchanged.
   - I also made one change that wasn't requested. When the output falls back to the raw stdout/stderr file and that file doesn't end in a newline, a newline is now added before the closing delimiter. Without it, the delimiter would be stuck to the end of the last line and the output block would be malformed.
2. **'Sanitize workflow name':** newlines are now stripped from the input before the sed/xargs filtering. Right before writing, the value goes through `printf '%s' "$SANITIZED" | tr -d '\n\r'` again, and `$GITHUB_OUTPUT` is now quoted. This is the same change the previous harden made.
3. **Unpinned actions:** `google-github-actions/auth@v3` is now pinned to `7c6bc770dae815cd3e89ee6cdf493a5fab2cc093` and `actions/upload-artifact@v6` to `b7c566a772e6b6bfb58ed0dc250532a479d7789f`. I resolved both commits with `lookup_action_sha` and kept the tag as a trailing comment. This replaces the old `# ratchet:exclude` comments, so the ratchet tool will now treat these lines like any other pinned action.
4. **Prior fix:** re-applied. Its only security change was the workflow-name sanitizing in item 2. The rest of that patch was blank-line removal, which I didn't redo. One hunk in the old patch (the start of `runs:`) appears to add lines that the patch text doesn't show, so I couldn't tell what they were or re-apply them.

The telemetry step still runs `otel/opentelemetry-collector-contrib:0.108.0` by tag rather than by digest. It wasn't among these findings, so I didn't change it.

### Iteration 2

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

Auth step: restored `if: ${{ inputs.gcp_workload_identity_provider != '' }}` (reconstructed from upstream, since the original text was lost) and `id: 'auth'`, and pinned to google-github-actions/auth@7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 (# v3, SHA checked by lookup). Run Gemini CLI step: put the random EOF_RESP assignment back on its own line before the multi-line gemini_response write, and removed the trailing `OF" >> "${GITHUB_OUTPUT}"` text. Edited lines were not re-read and the action was not run afterwards.

