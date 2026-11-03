<!-- markdownlint-disable -->

# Hardening Report: google-github-actions--run-gemini-cli/v0.1.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **google-github-actions--run-gemini-cli/v0.1.20** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Sanitize workflow name' writes a value derived from inputs.workflow_name (via env WORKFLOW_NAME) to $GITHUB_OUTPUT as `echo "gh_workflow_name=$SANITIZED" >> $GITHUB_OUTPUT`. The required newline-strip `printf '%s' ... | tr -d '\n\r'` is not applied right before the write. The sed/xargs pipeline is not the required sanitization, and xargs can emit more than one line when the input is very long.

Locations:

- `action.yml:186`

### github-env-injection (severity: high)

Step 'Run Gemini CLI' writes multi-line values to $GITHUB_OUTPUT between a FIXED heredoc delimiter: `echo "gemini_response<<EOF"` / `echo "${RESPONSE}"` (or `cat ${TEMP_STDOUT}`) / `echo "EOF"`, and the same for `gemini_errors<<EOF` with ERROR_JSON/TEMP_STDERR. These values come from Gemini CLI output, which is driven by the untrusted inputs.prompt and can be steered by an attacker. Output containing a line equal to 'EOF' ends the block early and injects extra outputs. The delimiter must be generated from a secure random source (e.g. /dev/urandom).

Locations:

- `action.yml:341`
- `action.yml:350`

### unpinned-uses (severity: high)

action.yml has composite steps whose uses: references are not pinned to a full commit SHA: `uses: 'google-github-actions/auth@v3' # ratchet:exclude` and `uses: 'actions/upload-artifact@v6' # ratchet:exclude`.

Locations:

- `action.yml:213`
- `action.yml:421`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 70ef1b1..c3f58e4 100644
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
+        SAFE_WORKFLOW_NAME=$(printf '%s' "${WORKFLOW_NAME}" | tr -d '\n\r')
+        SANITIZED=$(echo "${SAFE_WORKFLOW_NAME}" | sed 's/[^ a-zA-Z0-9-]//g' | xargs | tr ' ' '_' | tr '[:upper:]' '[:lower:]')
+        echo "gh_workflow_name=$SANITIZED" >> "$GITHUB_OUTPUT"
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

Pinned google-github-actions/auth to 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 (v3) and actions/upload-artifact to b7c566a772e6b6bfb58ed0dc250532a479d7789f (v6). The 'Sanitize workflow name' step now strips newlines (printf '%s' | tr -d '\n\r') from the input before running sed/xargs. It strips them again right before writing to the quoted "$GITHUB_OUTPUT" (the earlier harden's change, re-applied). In 'Run Gemini CLI', each of the gemini_response and gemini_errors outputs now gets its own random heredoc delimiter, built from /dev/urandom via od. Multi-line values are written unchanged.

### Iteration 2

**Fixes applied:** unpinned-uses, github-env-injection, invalid-yaml

**Notes:**

Pinned google-github-actions/auth@v3 to 7c6bc770dae815cd3e89ee6cdf493a5fab2cc093 and actions/upload-artifact@v6 to b7c566a772e6b6bfb58ed0dc250532a479d7789f. Fixed the invalid YAML: the earlier iteration had pasted a stray upload-artifact uses line into the GH_WORKFLOW_NAME env value. It is back to '${{ steps.sanitize_workflow_name.outputs.gh_workflow_name }}'. The 'Sanitize workflow name' step now runs printf '%s' | tr -d '\n\r' just before writing to "$GITHUB_OUTPUT", which is now quoted.

