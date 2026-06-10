<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.46.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.46.3** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: `action/entrypoint.sh` expands untrusted input-derived shell variables without double-quoting, allowing shell metacharacter injection.

1. Line 17: `if [[ -z $(ls ${TARGET} 2>/dev/null) ]]; then` — `TARGET` is set from `INPUT_FILES` (which maps to `inputs.files`). The unquoted `${TARGET}` in the `ls` command allows glob expansion and word-splitting on attacker-controlled input.

2. Line 53: `ARGS="${TARGET}"` — unquoted `TARGET` (from `inputs.files`) is assigned to ARGS, then ARGS is used unquoted in command execution on lines 72–73: `${COMMAND} ${ARGS} --format json | ...` and `${COMMAND} ${ARGS}`. An attacker-controlled `inputs.files` value containing shell metacharacters (`;`, `|`, `&`, etc.) can inject arbitrary commands.

3. Line 68: `ARGS+=" --config ${INPUT_CONFIG}"` — `INPUT_CONFIG` (from `inputs.config`) is appended to ARGS without quoting, and then ARGS is passed unquoted to the command on lines 72–73, enabling argument injection from attacker-controlled config path values.

All three cases should use double-quoted expansions: `"${TARGET}"`, `"${INPUT_CONFIG}"`, and `"${ARGS}"`.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:53`
- `action/entrypoint.sh:68`
- `action/entrypoint.sh:72`
- `action/entrypoint.sh:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in action/entrypoint.sh:
1. Line 17: Added double-quotes around `${TARGET}` in the `ls` command: `ls "${TARGET}"` — prevents glob expansion and word-splitting on attacker-controlled input.
2. Lines 53+: Converted ARGS from a plain string to a bash array (`ARGS=("${TARGET}")`). This is the idiomatic safe approach for accumulating shell arguments — each element is stored separately and never subject to word-splitting or glob expansion.
3. Line 68: Changed `ARGS+=" --config ${INPUT_CONFIG}"` to `ARGS+=("--config" "${INPUT_CONFIG}")` — the config path is now a properly quoted, separate array element.
4. Lines 72-73: Changed `${COMMAND} ${ARGS}` to `"${COMMAND}" "${ARGS[@]}"` — the array is expanded with double-quotes using `[@]` syntax, which preserves element boundaries and prevents metacharacter injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted uses of `${_INSTALL_DIR}` in `action/entrypoint.sh` by adding double quotes around each expansion:
- Line 43: `mkdir -p ${_INSTALL_DIR}` → `mkdir -p "${_INSTALL_DIR}"`
- Line 45: `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR} ${CMD_NAME}.exe` → `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}" ${CMD_NAME}.exe`
- Line 47: `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR} ./${CMD_NAME}` → `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}" ./${CMD_NAME}`

This prevents shell metacharacter interpretation when `runner.temp` (or any other value flowing through `INSTALL_DIR`) contains special characters.

