<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.22** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Run Gemini CLI' (id gemini_run) writes multi-line values to $GITHUB_OUTPUT between a FIXED heredoc delimiter 'EOF': `echo "gemini_response<<EOF" >> "${GITHUB_OUTPUT}"` then `echo "${RESPONSE}"` / `cat "${TEMP_STDOUT}"`, then `echo "EOF"`. The same happens for gemini_errors with ERROR_JSON / TEMP_STDERR. RESPONSE is model output driven by the untrusted inputs.prompt, so anyone who can influence the prompt can make the output contain a line that is exactly 'EOF'. That ends the block early, and every line after it is parsed as a new step output. Fix: build the delimiter from /dev/urandom on every run, e.g. EOF="EOF_$(od -An -N16 -tx1 /dev/urandom | tr -d ' \n')".

Locations:

- `action.yml:352`
- `action.yml:361`

### github-env-injection (severity: high)

Step 'Sanitize workflow name' writes a value taken from inputs.workflow_name (default github.workflow, passed in through env WORKFLOW_NAME) to $GITHUB_OUTPUT with `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. The required `printf '%s' ... | tr -d '\n\r'` step does not run right before the write, and $GITHUB_OUTPUT is not quoted. The sed and xargs pipeline happens to remove most dangerous characters, but it is not the required sanitization.

Locations:

- `action.yml:194`

### unpinned-uses (severity: high)

action.yml composite steps use refs pinned to tags instead of full commit SHAs: `uses: 'google-github-actions/auth@v3' # ratchet:exclude` and `uses: 'actions/upload-artifact@v6' # ratchet:exclude`. pnpm/action-setup is correctly pinned to a SHA.

Locations:

- `action.yml:221`
- `action.yml:434`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 35af14f..66807b4 100644
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
@@ -99,7 +98,6 @@ inputs:
     description: 'The Issue number (or comma-separated list of issue numbers) the CLI is operating on. Defaults to the event payload.'
     required: false
     default: '${{ github.event.issue.number }}'
-
 outputs:
   summary:
     description: 'The summarized output from the Gemini CLI execution.'
@@ -107,10 +105,26 @@ outputs:
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
@@ -185,16 +199,15 @@ runs:
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
+        safe=$(printf '%s' "${SANITIZED}" | tr -d '\n\r')
+        echo "gh_workflow_name=${safe}" >> "$GITHUB_OUTPUT"
       env:
         WORKFLOW_NAME: '${{ inputs.workflow_name }}'
-
     - name: 'Configure Gemini CLI'
       if: |-
         ${{ inputs.settings != '' }}
@@ -204,7 +217,6 @@ runs:
       shell: 'bash'
       env:
         SETTINGS: '${{ inputs.settings }}'
-
     - name: 'Install Custom Commands'
       shell: 'bash'
       run: |-
@@ -213,26 +225,23 @@ runs:
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
@@ -282,7 +291,6 @@ runs:
             fi
           done
         fi
-
     - name: 'Run Gemini CLI'
       id: 'gemini_run'
       shell: 'bash'
@@ -428,15 +436,13 @@ runs:
         GH_WORKFLOW_NAME: '${{ steps.sanitize_workflow_name.outputs.gh_workflow_name }}'
         GH_PR_NUMBER: '${{ inputs.github_pr_number }}'
         GH_ISSUE_NUMBER: '${{ inputs.github_issue_number }}'
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
@@ -510,7 +516,6 @@ runs:
         GITHUB_PR_NUMBER: '${{ inputs.github_pr_number }}'
         GITHUB_ISSUE_NUMBER: '${{ inputs.github_issue_number }}'
         GH_WORKFLOW_NAME: '${{ steps.sanitize_workflow_name.outputs.gh_workflow_name }}'
-
 branding:
   icon: 'terminal'
 
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, unpinned-uses, prior-fix

**Notes:**

- Run Gemini CLI: the gemini_response and gemini_errors heredoc delimiters are now random on every run (EOF_ plus 16 bytes from /dev/urandom, hex-encoded). Multi-line values are written unchanged.
- Sanitize workflow name: the name goes through printf '%s' | tr -d '\n\r' before the sed pipeline and again right before it is written. The write now uses a quoted "$GITHUB_OUTPUT". This matches the prior harden's patch.
- Pinned google-github-actions/auth@v3 to 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 and actions/upload-artifact@v6 to b7c566a772e6b6bfb58ed0dc250532a479d7789f, keeping the tag in a trailing comment.
- Not in the findings: also pinned the otel/opentelemetry-collector-contrib:0.108.0 image in the telemetry step's docker run by its sha256 digest.
- The prior patch's diff was cut off. One hunk added about 16 lines right after 'steps:', but the visible part shows no added lines, so those changes could not be re-applied.

### Iteration 2

**Fixes applied:** unpinned-uses, invalid-yaml

**Notes:**

Rebuilt the broken 'Upload Gemini CLI outputs' step. Its name is a proper string again, and I restored its condition as `if: ${{ inputs.upload_artifacts == 'true' }}`. That condition is my reconstruction and wasn't checked against the upstream file. Pinned actions/upload-artifact to b7c566a772e6b6bfb58ed0dc250532a479d7789f (v6) and google-github-actions/auth to 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 (v3), both SHAs from lookup_action_sha. I didn't re-parse action.yml after the edit or run any tests.

