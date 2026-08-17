<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.46.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.46.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, several env vars that hold workflow-controllable input values are expanded unquoted in shell commands, allowing shell metacharacter injection.

1. Line 17: `ls ${TARGET}` — TARGET is set from INPUT_FILES (which maps to inputs.files). An attacker-controlled value with spaces, globs, or semicolons will be word-split and interpreted by the shell.

2. Line 63: `ARGS+=" --config ${INPUT_CONFIG}"` — INPUT_CONFIG (from inputs.config) is unquoted inside the string, allowing embedded spaces or metacharacters to break argument boundaries.

3. Lines 66–67: `${COMMAND} ${ARGS}` — ARGS is unquoted when passed to the command, so any whitespace or shell metacharacters embedded in INPUT_FILES or INPUT_CONFIG (via ARGS) will be word-split and interpreted by the shell, enabling command injection.

All these env vars are set in action.yml from inputs.* expressions (e.g. INPUT_FILES: ${{ inputs.files }}, INPUT_CONFIG: ${{ inputs.config }}) and must be double-quoted wherever they are expanded in shell.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:63`
- `action/entrypoint.sh:66`
- `action/entrypoint.sh:67`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in action/entrypoint.sh:
1. Line 17: Quoted `${TARGET}` → `"${TARGET}"` in the `ls` call to prevent word-splitting and glob injection.
2. Lines 55–63: Converted `ARGS` from a plain string to a bash array (`ARGS=("${TARGET}")`). Each argument is now appended as a properly quoted array element (e.g., `ARGS+=("--config" "${INPUT_CONFIG}")`), preventing embedded spaces or metacharacters in INPUT_CONFIG from breaking argument boundaries.
3. Lines 66–67: Replaced unquoted `${COMMAND} ${ARGS}` with `"${COMMAND}" "${ARGS[@]}"` so the command and all its arguments are properly quoted when expanded, preventing word-splitting and shell metacharacter injection from INPUT_FILES or INPUT_CONFIG values.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 9 workflow files:

1. unpinned-uses: Pinned all 14 action references to their full 40-char SHA commits with tag comments preserved. Files updated: audit.yml, ci.yml, committed.yml, maturin.yml, post-release.yml, pre-commit.yml, rust-next.yml, template.yml, test-action.yml.

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. In post-release.yml: github.ref_name→REF_NAME, matrix.target→MATRIX_TARGET, matrix.os→MATRIX_OS, needs.create-release.outputs.tag→RELEASE_TAG. In template.yml: github.actor→ACTOR env var; TEMPLATE_URL/TEMPLATE_BRANCH are workflow-controlled env vars referenced as shell variables.

3. github-env-injection: All writes to $GITHUB_ENV in post-release.yml now sanitize values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing (TAG from github.ref_name, and ASSET values).

4. missing-permissions: Added `permissions: contents: read` to test-action.yml. maturin.yml already had a top-level permissions block; added `id-token: write` to the release job for PyPI publishing.

