<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v1.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Install mikefarah/yq' run: block directly interpolates ${{ steps.info.outputs.yq-installed }}, ${{ steps.info.outputs.bin-path }}, and ${{ steps.define-binary.outputs.name }} inside shell commands. These steps.*.outputs.* values are workflow-controllable and flow through YAML template substitution before the shell sees them, enabling command injection. Offending lines: `if [ "${{ steps.info.outputs.yq-installed }}" = "true" ]; then` and `yq_path="${{ steps.info.outputs.bin-path }}/${{ steps.define-binary.outputs.name }}"`.

Locations:

- `action.yml:75`
- `action.yml:78`

### script-injection (severity: high)

Rule (a): The 'Print yq version' run: block is entirely a ${{ }} expression: `run: ${{ steps.install-yq.outputs.yq-path }} --version`. The step output value is interpolated directly as a shell command, allowing an attacker who can influence step outputs to execute arbitrary commands.

Locations:

- `action.yml:87`

### script-injection (severity: high)

Rule (a): The 'Convert' run: block directly interpolates ${{ steps.install-yq.outputs.yq-path }} inside the shell script: `"${{ steps.install-yq.outputs.yq-path }}" \`. This step output is workflow-controllable and is interpolated before the shell parses the command, enabling command injection.

Locations:

- `action.yml:95`

### github-env-injection (severity: high)

The 'Install mikefarah/yq' run: block constructs yq_path from ${{ steps.info.outputs.bin-path }} and ${{ steps.define-binary.outputs.name }} (both workflow-controllable steps.*.outputs.* values) and then writes it to $GITHUB_OUTPUT via `echo "yq-path=${yq_path}" >> "$GITHUB_OUTPUT"` without applying the required sanitization (`printf '%s' ... | tr -d '\n\r'`). A newline injected into either step output could poison GITHUB_OUTPUT with attacker-controlled key-value pairs.

Locations:

- `action.yml:84`

### unpinned-uses (severity: high)

The composite action step 'Download mikefarah/yq' references `uses: robinraju/release-downloader@v1`, which is a mutable tag reference rather than a pinned 40-character commit SHA. If the tag is moved or the repository is compromised, the action will silently execute different code.

Locations:

- `action.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all 5 findings in hardened/action/action.yml:
1. Pinned robinraju/release-downloader@v1 to full SHA @28fc21f50d76778e7023361aa1f863e717d3d56f.
2. In 'Install mikefarah/yq': moved steps.info.outputs.yq-installed, steps.info.outputs.bin-path, and steps.define-binary.outputs.name into env vars (YQ_INSTALLED, BIN_PATH, BINARY_NAME) and replaced inline ${{ }} expressions in the shell script with those env vars.
3. Sanitized yq_path before writing to GITHUB_OUTPUT using printf '%s' | tr -d '\n\r'.
4. In 'Print yq version': moved steps.install-yq.outputs.yq-path to env var YQ_PATH; run: now executes "$YQ_PATH" --version.
5. In 'Convert': moved steps.install-yq.outputs.yq-path to env var YQ_PATH; shell script now references "${YQ_PATH}" instead of the inline expression.

### Iteration 2

**Fixes applied:** github-env-injection, invalid-yaml

**Notes:**

Fixed three findings: (1) src/collect-info.sh line 31 - sanitized bin_path before writing to GITHUB_OUTPUT using `printf '%s' "${bin_path}" | tr -d '\n\r'`; (2) src/convert.sh line 26 - sanitized result_path before writing to GITHUB_OUTPUT using `printf '%s' "${result_path}" | tr -d '\n\r'`; (3) action.yml line 99 - converted single-line `run: "$YQ_PATH" --version` to a block scalar `run: |\n  "$YQ_PATH" --version` to fix the YAML parse error.

