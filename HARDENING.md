<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in action.yml use mutable version tags instead of pinned full-length commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or compromised.

- `robinraju/release-downloader@v1` (line 65) — uses a mutable major-version tag
- `actions/github-script@v7` (line 89) — uses a mutable major-version tag

Both should be replaced with their corresponding 40-character hex commit SHAs.

Locations:

- `action.yml:65`
- `action.yml:89`

### script-injection (severity: high)

Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell command strings (rule a). GitHub Actions performs YAML template substitution before the shell ever sees the string, so any expression — including `steps.*.outputs.*` — can inject arbitrary shell metacharacters.

1. **"Check same from and to" step (~line 57):** `run: cp "${INPUT_INPUT}" "${{ steps.info.outputs.yq-temp-file-path }}"` — `${{ steps.info.outputs.yq-temp-file-path }}` is interpolated directly into the shell command.

2. **"Install mikefarah/yq" step (~line 68):** Three expressions are interpolated directly into shell arguments:
   - `"${{ steps.binary.outputs.name }}"`
   - `"${{ steps.info.outputs.bin-path }}"`
   - `"${{ steps.download-binary.outputs.tag_name }}"`

3. **"Convert" step (~line 84):** `> "${{ steps.info.outputs.yq-temp-file-path }}"` — the redirect target is directly interpolated.

Fix: move each `${{ steps.*.outputs.* }}` value into an `env:` variable and reference it as a quoted shell variable (e.g., `"$STEP_OUTPUT"`) inside the `run:` block.

Locations:

- `action.yml:57`
- `action.yml:68`
- `action.yml:84`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed two unpinned action references by pinning to full commit SHAs: robinraju/release-downloader@v1 → @28fc21f50d76778e7023361aa1f863e717d3d56f and actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b. Fixed script injection in three steps by moving all ${{ steps.*.outputs.* }} expressions into env: blocks and referencing them as shell variables (or process.env.VAR in JavaScript for the github-script step).

