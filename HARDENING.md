<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: The 'Print yq version' step uses the env var ${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} unquoted as a shell command. This variable holds `steps.install-yq.outputs.yq-path` (a steps.*.outputs.* value, which is untrusted per the check rules). An unquoted expansion allows shell metacharacters in the value to be interpreted by the shell. Offending line: `run: ${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version`

Locations:

- `action.yml:103`

### script-injection (severity: high)

Rule (b) violation: The 'Print resulting' step uses three unquoted shell variable expansions: ${STEP_CONVERT_OUTPUT_RESULT_PATH} (from steps.convert.outputs.result-path, a steps.*.outputs.* value), ${INPUT_FROM} (from inputs.from), and ${INPUT_TO} (from inputs.to). All are untrusted and unquoted, allowing shell metacharacters to be interpreted. Offending line: `run: cat ${STEP_CONVERT_OUTPUT_RESULT_PATH}/expected-${INPUT_FROM}.${INPUT_TO}`

Locations:

- `.github/actions/convert/action.yml:29`

### github-env-injection (severity: high)

The 'Install mikefarah/yq' step writes `echo "yq-path=${yq_path}" >> "$GITHUB_OUTPUT"` where yq_path is derived from env vars STEP_INFO_OUTPUT_BIN_PATH (steps.info.outputs.bin-path) and STEP_DEFINE_BINARY_OUTPUT_NAME (steps.define-binary.outputs.name) — both steps.*.outputs.* values, which are untrusted sources. No sanitization step (`printf '%s' ... | tr -d '\n\r'`) is applied before the write to $GITHUB_OUTPUT.

Locations:

- `action.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed three security findings:
1. script-injection (action.yml line 103): Quoted the STEP_INSTALL_YQ_OUTPUT_YQ_PATH variable when used as a command: changed `${STEP_INSTALL_YQ_OUTPUT_YQ_PATH} --version` to `"${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version`.
2. script-injection (.github/actions/convert/action.yml line 29): Quoted all variables in the cat command: changed `cat ${STEP_CONVERT_OUTPUT_RESULT_PATH}/expected-${INPUT_FROM}.${INPUT_TO}` to `cat "${STEP_CONVERT_OUTPUT_RESULT_PATH}/expected-${INPUT_FROM}.${INPUT_TO}"`.
3. github-env-injection (action.yml line 93): Added sanitization of yq_path before writing to GITHUB_OUTPUT: added `safe_yq_path="$(printf '%s' "${yq_path}" | tr -d '\n\r')"` and used `safe_yq_path` in the echo statement to strip any embedded newlines that could inject additional GITHUB_OUTPUT entries.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed the YAML parse error at line 99 in action.yml. The `run:` step for 'Print yq version' had a single-line value starting with a double-quoted string (`"${STEP_INSTALL_YQ_OUTPUT_YQ_PATH}" --version`), which YAML parsed as a complete quoted scalar and then rejected the trailing `--version`. Converted it to a block scalar using `run: |` so the entire command is treated as a literal string.

