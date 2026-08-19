<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ steps.* }}` expressions into shell commands (sub-rule a). YAML template substitution occurs before the shell parses the string, so any shell metacharacters in the substituted value are executed. Affected steps and offending lines:

1. "Check same from and to" step (line 63): `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.YQ_TEMP_FILE }}"`

2. "Install mikefarah/yq" step (lines 76–81): `mv ${{ steps.info.outputs.YQ_BINARY }} ${{ steps.info.outputs.YQ_EXEC }}` (also unquoted — sub-rule b), `chmod +x ${{ steps.info.outputs.YQ_EXEC }}`, and `echo "::debug::${{ steps.info.outputs.YQ_BINARY }}@..."`

3. "Convert" step (line 83): `run: ${{ steps.info.outputs.YQ_EXEC }} -P "$INPUTS_INPUT" ... > "${{ steps.info.outputs.YQ_TEMP_FILE }}"`

All `${{ ... }}` expressions must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g., `"$YQ_EXEC"`).

Locations:

- `action.yml:63`
- `action.yml:76`
- `action.yml:78`
- `action.yml:81`
- `action.yml:83`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised:

- `uses: robinraju/release-downloader@v1.9` (line 68) — tag `v1.9` is mutable
- `uses: actions/github-script@v7` (line 91) — tag `v7` is mutable

Each should be pinned to a full SHA, e.g. `uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7`.

Locations:

- `action.yml:68`
- `action.yml:91`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection by moving all ${{ steps.info.outputs.* }} expressions from run: shell strings into env: blocks and referencing them as double-quoted shell variables (${YQ_TEMP_FILE}, ${YQ_BINARY}, ${YQ_EXEC}, ${YQ_RELEASE}). Also fixed the Save output step by moving the YQ_TEMP_FILE expression to env: and accessing it via process.env.YQ_TEMP_FILE in JavaScript. Fixed unpinned-uses by pinning robinraju/release-downloader@v1.9 to SHA 368754b9c6f47c345fcfbf42bcb577c2f0f5f395 and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b, with the original tags preserved as comments.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed the YAML parsing error at line 100 in the 'Convert' step. The `run:` value started with a double-quoted string (`"${YQ_EXEC}"`), which YAML parsed as a complete quoted scalar and rejected the trailing shell arguments. Converted the single-line `run:` to a block scalar (`run: |`) so the entire shell command is treated as a literal string.

### Iteration 3

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 6 workflow files:

1. script-injection (functional-tests.yml): Moved all ${{ }} expressions from run: blocks to env: blocks. The critical `run: ${{ matrix.setup-command }}` was replaced with env: SETUP_COMMAND + `printf '%s' "$SETUP_COMMAND" | sh`. Other expressions (steps.convert.outputs.output, matrix.to) were moved to env: CONVERT_OUTPUT and MATRIX_TO respectively. Both convert and convert-container jobs were fixed.

2. unpinned-uses: Pinned all 11 action references to full 40-character SHA hashes with tag comments: actions/checkout@v4, fabasoad/reusable-workflows@main, simbo/changes-since-last-release-action@v1, softprops/action-gh-release@v2, fischerscode/tagger@v0, github/codeql-action/{init,analyze,upload-sarif}@v3, anchore/scan-action@v3, micnncim/action-label-syncer@v1, FantasticFiasco/action-update-license-year@v3.

3. missing-permissions: Added top-level permissions blocks to all 6 workflows with minimal required permissions: functional-tests.yml and linting.yml get `{}`, release.yml gets `contents: write`, security.yml gets `contents: read` + `security-events: write`, sync-labels.yml gets `issues: write`, update-license.yml gets `contents: write` + `pull-requests: write`.

