<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell commands, violating sub-rule (a). This allows an attacker-controlled value to be injected into the shell before quoting can protect it.

1. "Check same from and to" step (line 63): `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — `steps.info.outputs.YQ_TEMP_FILE` is interpolated directly into the shell command.

2. "Install mikefarah/yq" step (line 75): `mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}` — both step outputs are interpolated unquoted into the mv command.

3. "Install mikefarah/yq" step (line 77): `chmod +x ${{ steps.info.outputs.YQ_EXEC }}` — step output interpolated directly.

4. "Convert" step (line 83): `run: ${{ steps.info.outputs.YQ_EXEC }} -P "$INPUTS_INPUT" ... > "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — the entire command starts with a `${{ }}` expression, and another appears in the output redirect.

All `steps.*.outputs.*` values are workflow-controllable and must be passed via `env:` variables and double-quoted, never interpolated directly into `run:` scripts.

Locations:

- `action.yml:63`
- `action.yml:75`
- `action.yml:77`
- `action.yml:83`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

- Line 68: `uses: robinraju/release-downloader@v1.9` — mutable tag `v1.9`
- Line 86: `uses: actions/github-script@v7` — mutable tag `v7`

Both should be pinned to a full SHA, e.g. `uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7`.

Locations:

- `action.yml:68`
- `action.yml:86`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all four script-injection locations by moving ${{ steps.info.outputs.* }} expressions into env: blocks and referencing them as shell variables (${YQ_TEMP_FILE}, ${YQ_BINARY}, ${YQ_EXEC}). The 'Save output' step's JavaScript now reads the temp file path via process.env.YQ_TEMP_FILE instead of inline interpolation. Pinned robinraju/release-downloader@v1.9 to @368754b9c6f47c345fcfbf42bcb577c2f0f5f395 and actions/github-script@v7 to @f28e40c7f34bde8b3046d885e986cb6290c5673b, preserving original tags as comments.

### Iteration 2

**Fixes applied:** script-injection, invalid-yaml

**Notes:**

1. script-injection (line 80): Moved `${{ steps.yq.outputs.release }}` from the inline `run:` shell string into the step's `env:` block as `YQ_RELEASE: "${{ steps.yq.outputs.release }}"`, then referenced it as `${YQ_RELEASE}` in the shell command. This prevents any shell metacharacters in the release output from being interpreted. 2. invalid-yaml (line 99): Converted the single-line `run: "${YQ_EXEC}" -P ...` to a block scalar (`run: |`) so YAML correctly parses the entire command as a literal string rather than treating the leading quoted `"${YQ_EXEC}"` as a complete scalar and rejecting the trailing arguments.

