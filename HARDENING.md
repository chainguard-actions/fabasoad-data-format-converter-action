<!-- markdownlint-disable -->

# Hardening Report: fabasoad--data-format-converter-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fabasoad--data-format-converter-action/v1.0.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions inside shell commands, enabling script injection. (1) 'Install mikefarah/yq' step interpolates `${{ steps.info.outputs.yq-installed }}`, `${{ steps.info.outputs.bin-path }}`, and `${{ steps.define-binary.outputs.name }}` directly in the shell script. (2) 'Print yq version' step uses `run: ${{ steps.install-yq.outputs.yq-path }} --version` — the entire command is an expression. (3) 'Convert' step interpolates `${{ steps.install-yq.outputs.yq-path }}` directly in the shell script. These step outputs are workflow-controllable and must be passed via env vars with quoted expansions.

Locations:

- `action.yml:68`
- `action.yml:80`
- `action.yml:87`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in .github/workflows/functional-tests.yml directly interpolate `${{ ... }}` expressions inside shell commands. (1) 'Print converted file' step: `run: cat ${{ steps.convert.outputs.result-path }}/expected-${{ matrix.from }}.${{ matrix.to }}`. (2) 'Validate' step: `${{ matrix.to }}`, `${{ matrix.from }}`, and `${{ steps.convert.outputs.result-path }}` are interpolated directly in the run block. (3) 'Setup' step in convert-container job: `run: ${{ matrix.setup-command }}` — the entire run command is a matrix expression, allowing arbitrary shell command injection.

Locations:

- `.github/workflows/functional-tests.yml:38`
- `.github/workflows/functional-tests.yml:40`
- `.github/workflows/functional-tests.yml:65`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in .github/workflows/test-source-pattern.yml directly interpolate `${{ ... }}` expressions inside shell commands. (1) 'List resulting files' step: `run: ls -la ${{ steps.convert.outputs.result-path }}`. (2) 'Test action completion' step: `${{ steps.convert.outputs.result-path }}` and `${{ matrix.expected-files-amount }}` are interpolated directly in the run block.

Locations:

- `.github/workflows/test-source-pattern.yml:47`
- `.github/workflows/test-source-pattern.yml:49`

### github-env-injection (severity: high)

In action.yml, the 'Install mikefarah/yq' step constructs `yq_path` from `${{ steps.info.outputs.bin-path }}/${{ steps.define-binary.outputs.name }}` (step outputs that are workflow-controllable) and writes it to `$GITHUB_OUTPUT` via `echo "yq-path=${yq_path}" >> "$GITHUB_OUTPUT"` without applying the required sanitization (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject additional entries into the output file.

Locations:

- `action.yml:68`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks: action.yml: `robinraju/release-downloader@v1`; functional-tests.yml: `actions/checkout@v4`; linting.yml: `fabasoad/reusable-workflows/.github/workflows/wf-pre-commit.yml@main`; release.yml: `fabasoad/reusable-workflows/.github/workflows/wf-github-release.yml@main`; security.yml: `fabasoad/reusable-workflows/.github/workflows/wf-security-sast.yml@main`; sync-labels.yml: `fabasoad/reusable-workflows/.github/workflows/wf-sync-labels.yml@main`; test-source-pattern.yml: `actions/checkout@v4`; update-license.yml: `fabasoad/reusable-workflows/.github/workflows/wf-update-license.yml@main`.

Locations:

- `action.yml:57`
- `.github/workflows/functional-tests.yml:34`
- `.github/workflows/functional-tests.yml:68`
- `.github/workflows/linting.yml:12`
- `.github/workflows/release.yml:11`
- `.github/workflows/security.yml:14`
- `.github/workflows/sync-labels.yml:12`
- `.github/workflows/test-source-pattern.yml:38`
- `.github/workflows/update-license.yml:11`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning they run with the default (potentially broad) token permissions: functional-tests.yml, linting.yml, release.yml, sync-labels.yml, test-source-pattern.yml, update-license.yml. Each should declare minimal required permissions at the top level or per-job.

Locations:

- `.github/workflows/functional-tests.yml:1`
- `.github/workflows/linting.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sync-labels.yml:1`
- `.github/workflows/test-source-pattern.yml:1`
- `.github/workflows/update-license.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and .github/workflows/ files:

1. script-injection (action.yml): Moved ${{ steps.info.outputs.yq-installed }}, ${{ steps.info.outputs.bin-path }}, ${{ steps.define-binary.outputs.name }} into env vars in 'Install mikefarah/yq' step; moved ${{ steps.install-yq.outputs.yq-path }} into env var YQ_PATH for both 'Print yq version' and 'Convert' steps.

2. github-env-injection (action.yml): Added printf '%s' ... | tr -d '\n\r' sanitization before writing yq-path to $GITHUB_OUTPUT.

3. script-injection (functional-tests.yml): Moved result-path, matrix.from, matrix.to into env vars for Print/Validate steps; moved matrix.setup-command into env var and used sh -c for Setup step.

4. script-injection (test-source-pattern.yml): Moved result-path and expected-files-amount into env vars.

5. unpinned-uses: Pinned robinraju/release-downloader@v1 to SHA 28fc21f50d76778e7023361aa1f863e717d3d56f, actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262, and fabasoad/reusable-workflows@main to SHA 10062f8186847226cb4865efbb8047795d372bae across all workflow files.

6. missing-permissions: Added minimal permissions blocks to functional-tests.yml, linting.yml, release.yml, sync-labels.yml, test-source-pattern.yml, and update-license.yml.

### Iteration 2

**Fixes applied:** invalid-yaml

**Notes:**

Fixed invalid YAML at line 99 in action.yml. The `run: "${YQ_PATH}" --version` single-line value caused a YAML parse error because YAML interpreted `"${YQ_PATH}"` as a complete quoted scalar and then rejected the trailing `--version`. Converted to a block scalar (`run: |` with the command indented on the next line) so the entire command is treated as literal string content.

