<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.47.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.47.1** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: Unquoted shell variable expansion of untrusted data in action/entrypoint.sh. The variable TARGET (derived from INPUT_FILES, which comes from inputs.files) is expanded unquoted in `ls ${TARGET}` (line 17), allowing shell metacharacter injection. Similarly, INPUT_CONFIG is appended to ARGS without quoting (`ARGS+=" --config ${INPUT_CONFIG}"`, line ~60), and then ARGS is expanded unquoted in `${COMMAND} ${ARGS}` (lines ~63-64). An attacker-controlled input value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) could result in arbitrary command execution.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:60`
- `action/entrypoint.sh:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in hardened/action/action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` call (line 17): changed `ls ${TARGET}` to `ls "${TARGET}"` to prevent shell metacharacter injection from INPUT_FILES.
2. Converted ARGS from a plain string to a bash array: `ARGS=("${TARGET}")` instead of `ARGS="${TARGET}"`. Each flag is now appended as a separate array element (e.g., `ARGS+=("--config" "${INPUT_CONFIG}")`), ensuring INPUT_CONFIG and other inputs cannot inject shell metacharacters.
3. Expanded ARGS safely using `"${ARGS[@]}"` in both command invocations, and quoted the command itself as `"${COMMAND}"`, preventing word splitting and glob expansion on any user-supplied values.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 9 workflow files:

1. unpinned-uses: Pinned all 14 unique action references to full SHA commits across audit.yml, ci.yml, committed.yml, maturin.yml, post-release.yml, pre-commit.yml, rust-next.yml, template.yml, and test-action.yml.

2. script-injection: In post-release.yml, moved all ${{ }} expressions from run: blocks into step-level env: blocks (github.ref_name→REF_NAME, matrix.target→MATRIX_TARGET, matrix.os→MATRIX_OS, needs.create-release.outputs.tag→RELEASE_TAG). In template.yml, moved github.actor to ACTOR env var; TEMPLATE_URL and TEMPLATE_BRANCH were already workflow-level env vars and are now referenced as plain shell variables.

3. github-env-injection: In post-release.yml, the github.ref_name value is now sanitized with `printf '%s' "$REF_NAME" | tr -d '\n\r'` before writing to GITHUB_ENV. The staging variable (derived from matrix.target and release tag, both now in env vars) is also sanitized with `printf '%s' ... | tr -d '\n\r'` before writing ASSET to GITHUB_ENV.

4. missing-permissions: Added `permissions: contents: read` top-level block to test-action.yml.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions of `${_INSTALL_DIR}` in hardened/action/action/entrypoint.sh. Changed `mkdir -p ${_INSTALL_DIR}` to `mkdir -p "${_INSTALL_DIR}"`, `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR}` to `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}"`, and `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR}` to `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}"`. This prevents shell metacharacter interpretation when the runner.temp path (sourced from `${{ runner.temp }}` in action.yml) contains spaces or other special characters.

