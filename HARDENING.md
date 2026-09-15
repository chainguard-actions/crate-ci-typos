<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.50.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.50.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the env var INPUT_FILES (sourced from inputs.files via the env: block in action.yml) is assigned unquoted to TARGET on line 17 (`TARGET=${INPUT_FILES:-"."}`), then used unquoted in a shell expansion on line 18 (`ls ${TARGET}`). An attacker-controlled value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can cause command injection. Similarly, INPUT_CONFIG (sourced from inputs.config) is appended unquoted to ARGS on line 60 (`ARGS+=" --config ${INPUT_CONFIG}"`), and ARGS is then expanded unquoted on lines 63–64 (`${COMMAND} ${ARGS}`). All these env vars hold workflow-controllable data and must be double-quoted wherever they are expanded in the shell.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:18`
- `action/entrypoint.sh:60`
- `action/entrypoint.sh:63`
- `action/entrypoint.sh:64`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in hardened/action/action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` call (line 18): `ls "${TARGET}"`
2. Converted string-based ARGS variable to a bash array: `ARGS=("${TARGET}")` so each argument is a separate, properly-quoted element
3. Flags `--isolated` and `--write-changes` are now added as individual quoted array elements
4. `INPUT_CONFIG` is appended as two separate quoted elements `("--config" "${INPUT_CONFIG}")` preventing metacharacter injection
5. Command invocations now use `"${ARGS[@]}"` (proper quoted array expansion) instead of unquoted string `${ARGS}`
6. `${COMMAND}` and `${SOURCE_DIR}/format_gh.sh` are also double-quoted in the final execution lines

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted expansions of ${_INSTALL_DIR} in hardened/action/action/entrypoint.sh (lines 43, 45, 47). Changed `mkdir -p ${_INSTALL_DIR}` to `mkdir -p "${_INSTALL_DIR}"`, `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR}` to `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}"`, and `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR}` to `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}"`. This prevents word splitting on whitespace or shell metacharacters when the runner.temp path contains spaces.

