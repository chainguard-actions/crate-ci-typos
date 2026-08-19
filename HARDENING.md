<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.44.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.44.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the env var TARGET (set from INPUT_FILES, which holds the value of inputs.files) is expanded unquoted in multiple places: `ls ${TARGET}` (line 17), the error message `${TARGET}` (line 18), and `ARGS="${TARGET}"` (line 50). The final command `${COMMAND} ${ARGS}` also expands ARGS unquoted (line 62). An attacker-controlled value in inputs.files containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the intended command and execute arbitrary shell commands. Similarly, INPUT_CONFIG is appended to ARGS unquoted: `ARGS+=" --config ${INPUT_CONFIG}"` (line 57), allowing the same injection via inputs.config.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:18`
- `action/entrypoint.sh:50`
- `action/entrypoint.sh:57`
- `action/entrypoint.sh:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh by: (1) quoting TARGET in the ls check on line 17 (`ls "${TARGET}"`); (2) converting ARGS from a string variable to a bash array (`ARGS=("${TARGET}")`), which allows proper quoted expansion; (3) appending --config and INPUT_CONFIG as separate quoted array elements (`ARGS+=("--config" "${INPUT_CONFIG}")`); (4) expanding the command and args with proper quoting (`"${COMMAND}" "${ARGS[@]}"`). This prevents attacker-controlled values in inputs.files or inputs.config containing shell metacharacters from breaking out of the intended command.

