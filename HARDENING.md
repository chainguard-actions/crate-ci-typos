<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.46.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.46.2** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the env var INPUT_FILES (sourced from inputs.files, an untrusted caller-controlled value) is assigned to TARGET and then expanded unquoted in `ls ${TARGET}` (line 17) and later passed unquoted as part of ${ARGS} to the typos command (lines 63–64). Additionally, INPUT_CONFIG (from inputs.config) is appended to ARGS unquoted: `ARGS+=" --config ${INPUT_CONFIG}"` (line 59), and the entire ARGS variable is then expanded unquoted in `${COMMAND} ${ARGS}`. Unquoted expansion of these untrusted env vars allows shell metacharacter injection (e.g., semicolons, pipes, glob characters).

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:59`
- `action/entrypoint.sh:63`
- `action/entrypoint.sh:64`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` call (line 17) to prevent glob/metacharacter injection from INPUT_FILES.
2. Converted ARGS from a plain string to a bash array so each argument is stored as a discrete, properly-quoted element.
3. INPUT_CONFIG is now appended as a quoted array element: `ARGS+=("--config" "${INPUT_CONFIG}")` instead of unquoted string concatenation.
4. Command invocations use `"${ARGS[@]}"` (quoted array expansion) instead of unquoted `${ARGS}`, preventing word splitting and glob expansion on untrusted values.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted expansions of `${_INSTALL_DIR}` in action/entrypoint.sh (lines 43, 45, 47). Added double quotes around all three occurrences: `mkdir -p "${_INSTALL_DIR}"`, `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}" ...`, and `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}" ...`. The variable holds a value derived from `INSTALL_DIR` which is set to `${{ runner.temp }}` in action.yml, so it must be double-quoted in every shell expansion to prevent metacharacter interpretation.

