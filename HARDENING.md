<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--data-format-converter-action/v0.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ steps.*.outputs.* }}` expressions (rule a), which is a script-injection risk. Any expression inside a `run:` shell string undergoes YAML template substitution before the shell sees it, allowing injection of shell metacharacters.

- "Check same from and to" step: `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — `${{ steps.info.outputs.YQ_TEMP_FILE }}` is interpolated directly.
- "Install mikefarah/yq" step: `mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}` and `chmod +x ${{ steps.info.outputs.YQ_EXEC }}` — both unquoted and directly interpolated.
- "Convert" step: `run: ${{ steps.info.outputs.YQ_EXEC }} -P ...` and `> "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — expression used as the command itself and as a redirect target.
- "Save output" step: `fs.readFileSync('${{ steps.info.outputs.YQ_TEMP_FILE }}', ...)` — expression interpolated into a JS script string.

Locations:

- `action.yml:72`
- `action.yml:80`
- `action.yml:89`
- `action.yml:96`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: robinraju/release-downloader@v1` (tag `v1`)
- `uses: actions/github-script@v7` (tag `v7`)
These should be pinned to full SHA digests, e.g. `robinraju/release-downloader@<40-char-sha> # v1`.

Locations:

- `action.yml:78`
- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yml:

1. script-injection: Moved all ${{ steps.info.outputs.YQ_TEMP_FILE }}, ${{ steps.info.outputs.YQ_BINARY }}, and ${{ steps.info.outputs.YQ_EXEC }} expressions out of run: shell strings into env: blocks. Shell scripts now reference them as ${YQ_TEMP_FILE}, ${YQ_BINARY}, ${YQ_EXEC} environment variables. The JavaScript in the 'Save output' step now uses process.env.YQ_TEMP_FILE instead of interpolating the expression directly into the script string.

2. unpinned-uses: Pinned robinraju/release-downloader@v1 to @28fc21f50d76778e7023361aa1f863e717d3d56f # v1, and actions/github-script@v7 to @f28e40c7f34bde8b3046d885e986cb6290c5673b # v7.

### Iteration 2

**Fixes applied:** script-injection, invalid-yaml

**Notes:**

Fixed two issues in action.yml: (1) script-injection at line 95 — moved `${{ steps.yq.outputs.release }}` from the `run:` shell string into the step's `env:` block as `YQ_RELEASE`, then referenced it as `${YQ_RELEASE}` in the shell; (2) invalid-yaml at line 97 — converted the single-line `run: "${YQ_EXEC}" -P ...` (which YAML misparses because the value starts with a quote) into a block scalar `run: |` with the command on the next line.

