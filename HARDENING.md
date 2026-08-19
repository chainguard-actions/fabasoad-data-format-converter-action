<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v0.3.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in action.yml directly interpolate ${{ }} expressions, violating rule (a). (1) 'Check same from and to' step: `cp "${INPUT_INPUT}" "${{ steps.info.outputs.yq-temp-file-path }}"` — steps.*.outputs.* interpolated directly in shell. (2) 'Install mikefarah/yq' step: `"${{ steps.binary.outputs.name }}"`, `"${{ steps.info.outputs.bin-path }}"`, `"${{ steps.download-binary.outputs.tag_name }}"` all interpolated directly in the run: shell command. (3) 'Convert' step: `> "${{ steps.info.outputs.yq-temp-file-path }}"` interpolated directly in the run: shell command. These values flow through YAML template substitution before the shell sees them, enabling injection.

Locations:

- `action.yml:55`
- `action.yml:67`
- `action.yml:86`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/functional-tests.yml directly interpolate ${{ }} expressions, violating rule (a). (1) 'Save result' step (convert job): `echo '${{ steps.convert.outputs.output }}' > actual.${{ matrix.to }}` — both steps.*.outputs.* and matrix.* interpolated directly. (2) 'Print actual file' step: `cat actual.${{ matrix.to }}` — matrix.* interpolated directly. (3) 'Validate' step: `${{ matrix.to }}` used in file paths inside run:. (4) 'Clean up' step: `rm -f actual.${{ matrix.to }}`. (5) 'Setup' step (convert-container job): `run: ${{ matrix.setup-command }}` — the entire run: value is a matrix expression, allowing arbitrary shell command injection. (6) 'Save result' step (convert-container job): same pattern as (1).

Locations:

- `.github/workflows/functional-tests.yml:38`
- `.github/workflows/functional-tests.yml:40`
- `.github/workflows/functional-tests.yml:42`
- `.github/workflows/functional-tests.yml:46`
- `.github/workflows/functional-tests.yml:68`
- `.github/workflows/functional-tests.yml:80`

### github-env-injection (severity: high)

In src/install-yq.sh (called from the 'Install mikefarah/yq' step in action.yml), the variable `bin_path` — which is passed in as `$2` from `${{ steps.info.outputs.bin-path }}` (a steps.*.outputs.* value, treated as untrusted per the check rules) — is written directly to $GITHUB_PATH without sanitization: `echo "${bin_path}" >> "$GITHUB_PATH"`. The required sanitization step (`printf '%s' "$bin_path" | tr -d '\n\r'`) is absent.

Locations:

- `src/install-yq.sh:14`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branch names instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks: (1) action.yml: `uses: robinraju/release-downloader@v1` (tag); (2) action.yml: `uses: actions/github-script@v7` (tag); (3) functional-tests.yml: `uses: actions/checkout@v4` (tag); (4) linting.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main` (branch); (5) release.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main` (branch); (6) sync-labels.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main` (branch); (7) update-license.yml: `uses: fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main` (branch).

Locations:

- `action.yml:73`
- `action.yml:93`
- `.github/workflows/functional-tests.yml:35`
- `.github/workflows/linting.yml:12`
- `.github/workflows/release.yml:11`
- `.github/workflows/sync-labels.yml:12`
- `.github/workflows/update-license.yml:11`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions: functional-tests.yml, linting.yml, release.yml, sync-labels.yml, and update-license.yml. Each should declare minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/update-license.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 6 files:

1. script-injection (action.yml): Moved ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks for 'Check same from and to', 'Install mikefarah/yq', and 'Convert' steps.

2. script-injection (functional-tests.yml): Moved all ${{ matrix.* }} and ${{ steps.*.outputs.* }} expressions to env: blocks. The 'Setup' step that had `run: ${{ matrix.setup-command }}` was fixed by putting the command in SETUP_COMMAND env var and piping through `sh`.

3. github-env-injection (src/install-yq.sh): Added sanitization of bin_path before writing to GITHUB_PATH: `safe_bin_path=$(printf '%s' "${bin_path}" | tr -d '\n\r')`.

4. unpinned-uses: Pinned all mutable references to full commit SHAs: robinraju/release-downloader@v1→28fc21f50d76778e7023361aa1f863e717d3d56f, actions/github-script@v7→f28e40c7f34bde8b3046d885e986cb6290c5673b, actions/checkout@v4→11d5960a326750d5838078e36cf38b85af677262, fabasoad/reusable-workflows@main→5ebe0938b8d8ef97bbb051004c511ba78449a866.

5. missing-permissions: Added minimal permissions blocks to functional-tests.yml (contents: read), linting.yml (contents: read), release.yml (contents: write), sync-labels.yml (issues: write, pull-requests: write), and update-license.yml (contents: write).

