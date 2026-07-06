<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action--/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--data-format-converter-action--/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): In the 'Print yq version' step, the env var STEP_INSTALL_YQ_OUTPUT_YQ_PATH (sourced from ${{ steps.install-yq.outputs.yq-path }}, a workflow-controllable step output) is used **unquoted** as a shell command: `run: ${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version`. An unquoted expansion allows the shell to parse metacharacters (`;`, `|`, `&`, `$(...)`, etc.) from the value, enabling command injection.

Locations:

- `action.yml:97`

### github-env-injection (severity: high)

In the 'Install mikefarah/yq' step, the variable yq_path is constructed from STEP_INFO_OUTPUT_BIN_PATH and STEP_DEFINE_BINARY_OUTPUT_NAME (both sourced from prior step outputs, which are workflow-controllable via ${{ steps.info.outputs.bin-path }} and ${{ steps.define-binary.outputs.name }}) and then written directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:91`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two high-severity findings in hardened/action/action.yml:
1. script-injection (line 97): Quoted the STEP_INSTALL_YQ_OUTPUT_YQ_PATH variable in the 'Print yq version' step run command from `${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version` to `"${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version` to prevent shell metacharacter injection.
2. github-env-injection (line 91): Added newline sanitization before writing yq_path to GITHUB_OUTPUT. The value is now passed through `printf '%s' "${yq_path}" | tr -d '\n\r'` to strip embedded newlines before the echo to GITHUB_OUTPUT, preventing injection of arbitrary key=value pairs.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, invalid-yaml

**Notes:**

Fixed all three findings: (1) unpinned-uses: Pinned actions/checkout@v7 to SHA 9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 in functional-tests.yml and test-source-pattern.yml; pinned fabasoad/reusable-workflows@main to SHA 3ae541f80e3e6aca1f2c3705477fbef2890f1c98 in linting.yml, release.yml, security.yml, sync-labels.yml, and update-license.yml. (2) script-injection: In functional-tests.yml, moved matrix.setup-command into an env var (SETUP_COMMAND) and referenced it as $SETUP_COMMAND in the run script. In test-source-pattern.yml, double-quoted the ${STEP_CONVERT_OUTPUT_RESULT_PATH} variable in the ls command, and moved matrix.expected-files-amount into an env var (EXPECTED_FILES_AMOUNT) instead of direct interpolation. (3) invalid-yaml: Fixed action.yml line 99 where `run: "${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version` was invalid YAML (quoted scalar followed by trailing text); converted to a block scalar with `run: |`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/.github/workflows/functional-tests.yml (line 131): Changed `run: $SETUP_COMMAND` to `run: sh -c "$SETUP_COMMAND"` to quote the variable and prevent unquoted shell expansion of the matrix.setup-command value.
2. hardened/action/.github/actions/convert/action.yml (line 26): Added double quotes around the cat command path `"${STEP_CONVERT_OUTPUT_RESULT_PATH}/expected-${INPUT_FROM}.${INPUT_TO}"` to prevent unquoted variable expansions from allowing shell metacharacter interpretation.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in the 'Setup' step of the 'convert-container' job in .github/workflows/functional-tests.yml. Replaced `sh -c "$SETUP_COMMAND"` with `printf '%s\n' "$SETUP_COMMAND" | xargs sh -c 'exec "$@"' --`. This prevents shell injection because xargs splits the command by whitespace into individual tokens without interpreting shell metacharacters (like semicolons, pipes, etc.), and the shell script passed to sh -c is a literal hardcoded string ('exec "$@"') rather than the user-controlled value. The matrix-controlled tokens become positional parameters executed as a command, not as a shell script string.

