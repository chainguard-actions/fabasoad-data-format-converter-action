<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action--/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--data-format-converter-action--/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): In the 'Print yq version' step, the env var STEP_INSTALL_YQ_OUTPUT_YQ_PATH (which holds ${{ steps.install-yq.outputs.yq-path }}, a workflow-controllable steps.*.outputs.* value) is expanded **unquoted** in the run: block: `run: ${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version`. An unquoted shell variable expansion allows shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) embedded in the value to be interpreted by the shell, enabling command injection.

Locations:

- `action.yml:103`

### github-env-injection (severity: high)

In the 'Install mikefarah/yq' step, the variable yq_path is constructed from env vars STEP_INFO_OUTPUT_BIN_PATH and STEP_DEFINE_BINARY_OUTPUT_NAME, which hold ${{ steps.info.outputs.bin-path }} and ${{ steps.define-binary.outputs.name }} respectively (steps.*.outputs.* — untrusted per the check rules). The value is then written to $GITHUB_OUTPUT via `echo "yq-path=${yq_path}" >> "$GITHUB_OUTPUT"` without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline embedded in the value could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `action.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings in hardened/action/action.yml:
1. script-injection (line 103): Quoted the variable expansion in the 'Print yq version' step: changed `run: ${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version` to `run: "${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version`. This prevents shell metacharacters embedded in the yq-path output value from being interpreted as shell commands.
2. github-env-injection (line 96): Added newline sanitization before writing yq_path to $GITHUB_OUTPUT in the 'Install mikefarah/yq' step. The value is now sanitized via `safe_yq_path="$(printf '%s' "${yq_path}" | tr -d '\n\r')"` before being written with `echo "yq-path=${safe_yq_path}" >> "$GITHUB_OUTPUT"`, preventing newline injection of additional key=value pairs into GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, invalid-yaml

**Notes:**

Fixed all three findings: (1) unpinned-uses: Pinned actions/checkout@v7 to SHA 9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 in functional-tests.yml (twice) and test-source-pattern.yml; pinned fabasoad/reusable-workflows@main to SHA 3ae541f80e3e6aca1f2c3705477fbef2890f1c98 in linting.yml, release.yml, security.yml, sync-labels.yml, and update-license.yml. (2) script-injection: Moved matrix.setup-command to env block (SETUP_COMMAND) in functional-tests.yml Setup step; moved matrix.expected-files-amount to env block (EXPECTED_FILES_AMOUNT) in test-source-pattern.yml Test action completion step; added double-quotes around ${STEP_CONVERT_OUTPUT_RESULT_PATH} in the ls command in test-source-pattern.yml. (3) invalid-yaml: Converted the single-line run: "${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version to a block scalar (run: |) in action.yml to fix the YAML parse error.

