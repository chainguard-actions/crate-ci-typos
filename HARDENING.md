<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.51.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.51.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansions of untrusted input data in action/entrypoint.sh. The env vars INPUT_FILES and INPUT_CONFIG are set from `inputs.files` and `inputs.config` (workflow-controllable) and are used unquoted in shell commands, allowing shell metacharacter injection:
- Line 17: `ls ${TARGET}` — TARGET is derived from INPUT_FILES (unquoted, allows glob/word-splitting)
- Line 62: `ARGS+=" --config ${INPUT_CONFIG}"` — INPUT_CONFIG appended to ARGS unquoted
- Lines 65–66: `${COMMAND} ${ARGS}` — ARGS (containing untrusted TARGET and INPUT_CONFIG) expanded unquoted, allowing argument injection and shell metacharacter abuse

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:62`
- `action/entrypoint.sh:65`
- `action/entrypoint.sh:66`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/action/entrypoint.sh by: (1) quoting TARGET in the ls command on line 17 (`ls "${TARGET}"`); (2) converting ARGS from a string to a bash array (`ARGS=("${TARGET}")`), so each argument is a properly quoted separate element; (3) appending --config and INPUT_CONFIG as separate quoted array elements (`ARGS+=("--config" "${INPUT_CONFIG}")`); (4) using `"${ARGS[@]}"` expansion instead of unquoted `${ARGS}` when invoking the command, preventing shell metacharacter injection from INPUT_FILES and INPUT_CONFIG.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted expansions of ${_INSTALL_DIR} in hardened/action/action/entrypoint.sh. Changed `mkdir -p ${_INSTALL_DIR}`, `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR} ${CMD_NAME}.exe`, and `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR} ./${CMD_NAME}` to use double-quoted `"${_INSTALL_DIR}"` in all three places. This prevents word splitting and glob expansion on the value derived from ${{ runner.temp }}.

