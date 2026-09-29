<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character SHA digests, creating a supply-chain risk where the referenced action could be silently replaced:
- `robinraju/release-downloader@v1` (line 78) — tag `v1` is mutable
- `actions/github-script@v7` (line 104) — tag `v7` is mutable

These should be pinned to full SHA digests, e.g. `robinraju/release-downloader@<40-char-sha>` and `actions/github-script@<40-char-sha>`.

Locations:

- `action.yml:78`
- `action.yml:104`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). YAML template substitution occurs before the shell parses the string, so any expression value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can alter the command being executed.

**Step "Check same from and to" (line 74):**
`run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"`
The step-output expression is interpolated directly into the shell command.

**Step "Install mikefarah/yq" (lines 87, 89, 92):**
`mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}`
`chmod +x ${{ steps.info.outputs.YQ_EXEC }}`
`echo "::debug::${{ steps.info.outputs.YQ_BINARY }}@${{ steps.yq.outputs.release }} has been installed"`
Multiple step-output expressions are interpolated directly and unquoted into shell commands.

**Step "Convert" (line 100):**
`run: ${{ steps.info.outputs.YQ_EXEC }} -P "$INPUTS_INPUT" -p "$INPUTS_FROM" -o "$INPUTS_TO" > "${{ steps.info.outputs.YQ_TEMP_FILE }}"`
A step-output expression is used as the command name itself, and another as the output redirect target.

Fix: move all `${{ steps.info.outputs.* }}` values into `env:` variables and reference them as quoted shell variables (e.g. `"$YQ_EXEC"`, `"$YQ_BINARY"`, `"$YQ_TEMP_FILE"`).

Locations:

- `action.yml:74`
- `action.yml:87`
- `action.yml:89`
- `action.yml:92`
- `action.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two unpinned-uses findings by pinning robinraju/release-downloader@v1 to SHA 28fc21f50d76778e7023361aa1f863e717d3d56f and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b. Fixed five script-injection locations by moving all ${{ steps.info.outputs.* }} expressions into env: blocks in their respective steps ('Check same from and to', 'Install mikefarah/yq', 'Convert', and 'Save output'), then referencing them as shell environment variables or via process.env in the JavaScript script.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed the YAML parse error at line 98 in the 'Convert' step. The `run:` value started with a double-quoted string (`"${YQ_EXEC}"`), which YAML parsed as a complete quoted scalar and rejected the trailing shell arguments. Converted it to a block scalar (`run: |`) so the entire command is treated as a literal string by the YAML parser.

