<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.47.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.47.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation in action/entrypoint.sh: Multiple env vars holding workflow-controllable input values are expanded unquoted in shell commands.

1. `TARGET=${INPUT_FILES:-"."}` (INPUT_FILES comes from inputs.files) is then used unquoted: `ls ${TARGET}` (line 16) and `ARGS="${TARGET}"` (line 43), after which `${COMMAND} ${ARGS}` is executed with ARGS unquoted (line 57-58). An attacker-supplied `inputs.files` value containing shell metacharacters (`;`, `|`, `&`, glob chars, whitespace) will be split and interpreted by the shell.

2. `ARGS+=" --config ${INPUT_CONFIG}"` (line 53) — INPUT_CONFIG comes from inputs.config and is appended to ARGS unquoted, then executed unquoted as `${COMMAND} ${ARGS}`.

All these expansions should be double-quoted: `"${TARGET}"`, `"${INPUT_CONFIG}"`, and the final command should use an array or properly quoted form rather than unquoted `${ARGS}`.

Locations:

- `action/entrypoint.sh:16`
- `action/entrypoint.sh:43`
- `action/entrypoint.sh:53`
- `action/entrypoint.sh:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh by: (1) quoting `${TARGET}` in the `ls` call on line 16; (2) converting the ARGS variable from a plain string to a bash array (`ARGS=("${TARGET}")`), so each argument is stored as a discrete element; (3) appending `--config` and `${INPUT_CONFIG}` as separate quoted array elements (`ARGS+=("--config" "${INPUT_CONFIG}")`); (4) expanding the array with `"${ARGS[@]}"` in both command invocations so word-splitting and glob expansion cannot occur on attacker-controlled values.

