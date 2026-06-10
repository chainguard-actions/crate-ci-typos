<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.46.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.46.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, user-controlled inputs are expanded unquoted in shell commands, allowing shell metacharacter injection.

1. Line 17: `if [[ -z $(ls ${TARGET} 2>/dev/null) ]]; then` — `TARGET` is set from `INPUT_FILES` (which comes from `inputs.files`, a user-controlled input). The unquoted `${TARGET}` in the `ls` command allows glob expansion and word-splitting.

2. Line ~59: `ARGS+=" --config ${INPUT_CONFIG}"` — `INPUT_CONFIG` comes from `inputs.config` (user-controlled). The unquoted expansion allows word-splitting and metacharacter injection into the ARGS string.

3. Lines ~63-64: `${COMMAND} ${ARGS} --format json | ...` and `${COMMAND} ${ARGS}` — `ARGS` contains user-controlled data (TARGET from inputs.files, and optionally --config from inputs.config). The unquoted `${ARGS}` undergoes word-splitting, allowing an attacker to inject additional shell arguments or flags.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:59`
- `action/entrypoint.sh:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all three script injection vulnerabilities in action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` command on line 17 to prevent glob expansion and word-splitting.
2. Converted `ARGS` from a plain string to a bash array, so user-controlled values (TARGET from inputs.files, INPUT_CONFIG from inputs.config) are stored as discrete array elements rather than concatenated into a string subject to word-splitting.
3. Updated all command invocations to use `"${ARGS[@]}"` (quoted array expansion) so each argument is passed safely without word-splitting or metacharacter injection.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted `${_INSTALL_DIR}` variable expansions in action/entrypoint.sh by wrapping them in double quotes: `mkdir -p "${_INSTALL_DIR}"` (line 37), `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}" ${CMD_NAME}.exe` (line 39), and `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}" ./${CMD_NAME}` (line 41). This prevents shell metacharacter injection from the INSTALL_DIR environment variable which is set from `${{ runner.temp }}` in action.yml.

