<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell commands (sub-rule a). Any expression interpolated before the shell sees the string allows template-substitution injection.

1. "Check same from and to" step (line 62): `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"` — the step output is interpolated directly into the shell command.

2. "Install mikefarah/yq" step (lines 71–75): `mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}` and `chmod +x ${{ steps.info.outputs.YQ_EXEC }}` — step outputs used as unquoted shell arguments.

3. "Convert" step (line 82): `run: ${{ steps.info.outputs.YQ_EXEC }} -P "$INPUTS_INPUT" ...` — a step output is used as the command itself, and `"${{ steps.info.outputs.YQ_TEMP_FILE }}"` is interpolated as a redirect target.

Locations:

- `action.yml:62`
- `action.yml:71`
- `action.yml:82`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised.

- Line 65: `uses: robinraju/release-downloader@v1` (tag `v1`)
- Line 84: `uses: actions/github-script@v7` (tag `v7`)

Both should be pinned to their full SHA digests, e.g. `uses: actions/github-script@60a0d83039f74a4aee543508d2ffcb1c3799cdea # v7`.

Locations:

- `action.yml:65`
- `action.yml:84`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all 3 script-injection locations by moving ${{ steps.info.outputs.* }} expressions into env: blocks and referencing them as plain shell variables. Fixed 2 unpinned-uses by pinning robinraju/release-downloader@v1 to SHA 28fc21f50d76778e7023361aa1f863e717d3d56f and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b. Also fixed the Save output step to use process.env.YQ_TEMP_FILE instead of interpolating the step output directly into the JavaScript string.

### Iteration 2

**Fixes applied:** invalid-yaml, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) action.yml line 98 invalid YAML: converted single-line `run: "$YQ_EXEC" ...` to block scalar `run: |` to prevent YAML from misinterpreting the leading quoted string. (2) script-injection in functional-tests.yml: moved all ${{ steps.convert.outputs.output }}, ${{ matrix.to }}, and ${{ matrix.setup-command }} expressions out of run: shells into env: blocks, referencing them as plain env vars ($CONVERT_OUTPUT, $MATRIX_TO, $SETUP_COMMAND) — fixed in both convert and convert-container jobs. (3) unpinned-uses: pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in functional-tests.yml, and pinned all fabasoad/reusable-workflows @main references to SHA c5bd8945762dab6d2f5168b65f10355887ea40a3 in linting.yml, release.yml, security.yml, sync-labels.yml, and update-license.yml. (4) missing-permissions: added `permissions: {}` top-level block to functional-tests.yml, linting.yml, release.yml, sync-labels.yml, and update-license.yml (security.yml already had job-level permissions).

