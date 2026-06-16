<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--data-format-converter-action/v0.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ }}` expressions are directly interpolated inside `run:` shell command strings, allowing expression values to be parsed as shell code before the shell ever sees them.

• "Check same from and to" step (line 74): `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — the step-output value is injected directly into the shell command.

• "Install mikefarah/yq" step (lines 87, 89, 92): `mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}`, `chmod +x ${{ steps.info.outputs.YQ_EXEC }}`, and `echo "::debug::${{ steps.info.outputs.YQ_BINARY }}@${{ steps.yq.outputs.release }} has been installed"` — step outputs injected directly into shell commands, also unquoted.

• "Convert" step (line 100): `run: ${{ steps.info.outputs.YQ_EXEC }} -P "$INPUTS_INPUT" ... > "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — step outputs used as the command name and as a redirection target, both injected directly into the shell string.

All `${{ steps.*.outputs.* }}` values are workflow-controllable and must be passed via `env:` variables and double-quoted, never interpolated directly into `run:` scripts.

Locations:

- `action.yml:74`
- `action.yml:87`
- `action.yml:89`
- `action.yml:92`
- `action.yml:100`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

• `uses: robinraju/release-downloader@v1.9` (line 78) — mutable tag `v1.9`
• `uses: actions/github-script@v7` (line 104) — mutable tag `v7`

Both should be pinned to their full SHA digests, e.g. `uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7`.

Locations:

- `action.yml:78`
- `action.yml:104`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all 5 script-injection locations by moving ${{ steps.info.outputs.* }} expressions into env: blocks and referencing them as double-quoted shell variables. Fixed 2 unpinned-uses by pinning robinraju/release-downloader@v1.9 to SHA 368754b9c6f47c345fcfbf42bcb577c2f0f5f395 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b. Also updated the 'Save output' step's JavaScript to use process.env.YQ_TEMP_FILE instead of directly interpolating the step output into the script string.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed the invalid YAML in the 'Convert' step of action.yml. The `run:` field had a single-line value starting with a quoted string (`"$YQ_EXEC" -P ...`), which YAML incorrectly parsed as a complete quoted scalar and then rejected the trailing arguments. Converted it to a block scalar using `run: |` so the entire shell command is treated as a literal string.

