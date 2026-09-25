<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.50.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.50.3** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, env vars that hold untrusted inputs.* values (set via the action.yml env: block) are expanded unquoted in shell commands, allowing shell metacharacter injection. Specifically:
- Line 17: `ls ${TARGET}` — TARGET is derived from INPUT_FILES (inputs.files), unquoted, enabling glob/word-splitting attacks.
- Line 49: `ARGS="${TARGET}"` — unquoted TARGET appended into ARGS.
- Line 61: `ARGS+=" --config ${INPUT_CONFIG}"` — INPUT_CONFIG (inputs.config) is unquoted, allowing injection of extra shell arguments.
- Lines 64–65: `${COMMAND} ${ARGS}` — ARGS (containing attacker-controlled data from inputs.files and inputs.config) is unquoted, allowing word-splitting and shell metacharacter injection into the command invocation.
All these variables should be double-quoted (e.g., "${TARGET}", "${INPUT_CONFIG}", "${ARGS}") to prevent injection.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:49`
- `action/entrypoint.sh:61`
- `action/entrypoint.sh:64`
- `action/entrypoint.sh:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in hardened/action/action/entrypoint.sh by: (1) quoting ${TARGET} in the ls check on line 17; (2) converting ARGS from a string to a bash array (ARGS=("${TARGET}")), so each argument is a separate properly-quoted element; (3) quoting ${INPUT_CONFIG} and adding it as a separate array element alongside --config; (4) using "${ARGS[@]}" (array expansion) when invoking ${COMMAND}, ensuring all arguments remain properly quoted and word-splitting/metacharacter injection is prevented.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted `${_INSTALL_DIR}` variable expansions in `hardened/action/action/entrypoint.sh`: (1) `mkdir -p ${_INSTALL_DIR}` → `mkdir -p "${_INSTALL_DIR}"`, (2) `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR}` → `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}"`, and (3) `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR}` → `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}"`. The `INSTALL_DIR` env var is derived from `${{ runner.temp }}` in action.yml and flows into the shell unquoted, so double-quoting prevents shell metacharacter injection.

