<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fabasoad--data-format-converter-action/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a), allowing script injection if any upstream step output contains shell metacharacters.

1. "Check same from and to" step: `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.yq-temp-file-path }}"` — the step output is interpolated directly into the shell command.

2. "Install mikefarah/yq" step: the run block passes `"${{ steps.binary.outputs.name }}"`, `"${{ steps.info.outputs.bin-path }}"`, and `"${{ steps.download-binary.outputs.tag_name }}"` directly as shell arguments.

3. "Convert" step: `> "${{ steps.info.outputs.yq-temp-file-path }}"` is directly interpolated in the run block.

4. "Save output" step: `fs.readFileSync('${{ steps.info.outputs.yq-temp-file-path }}', ...)` is directly interpolated inside the JavaScript script block.

All `${{ ... }}` expressions inside `run:` blocks are expanded by the Actions template engine before the shell ever sees them, so any value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can execute arbitrary commands. These should be moved to `env:` variables and referenced via quoted `"$VAR"` shell expansions.

Locations:

- `action.yml:60`
- `action.yml:78`
- `action.yml:79`
- `action.yml:80`
- `action.yml:96`
- `action.yml:104`

### unpinned-uses (severity: high)

Two `uses:` references in action.yml are pinned to mutable version tags rather than immutable 40-character commit SHA digests. This means a compromised or force-pushed tag could silently substitute malicious code into the action.

- `uses: robinraju/release-downloader@v1` (line 67) — mutable tag `v1`
- `uses: actions/github-script@v7` (line 101) — mutable tag `v7`

These should be pinned to their full SHA-256 commit hashes, e.g. `uses: actions/github-script@60a0d83039c74a4aee543508d2ffcb1c3799cdea # v7`.

Locations:

- `action.yml:67`
- `action.yml:101`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all 4 script-injection locations by moving ${{ ... }} expressions into env: blocks and referencing them via shell/JS env vars. Pinned robinraju/release-downloader@v1 to SHA 28fc21f50d76778e7023361aa1f863e717d3d56f and actions/github-script@v7 to SHA f28e40c7f34bde8b3046d885e986cb6290c5673b.

