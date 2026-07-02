<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.48.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.48.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the user-controlled env var $INPUT_FILES (sourced from inputs.files) is assigned to $TARGET and then used unquoted in `ls ${TARGET}` (line 17) and later in `ARGS="${TARGET}"` (line 63). The $ARGS variable is subsequently used unquoted in the final command executions on lines 81-82: `${COMMAND} ${ARGS} --format json | ...` and `${COMMAND} ${ARGS}`. An attacker-controlled value in inputs.files containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can cause command injection. Similarly, $INPUT_CONFIG is appended to $ARGS unquoted on line 77 (`ARGS+=" --config ${INPUT_CONFIG}"`), allowing the same attack via inputs.config. All these env vars hold values from action inputs set in action.yml's env: block and must be double-quoted when used in shell expansions.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:77`
- `action/entrypoint.sh:81`
- `action/entrypoint.sh:82`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` check on line 17: `ls "${TARGET}"`
2. Converted the `ARGS` string variable to a bash array (`ARGS=("${TARGET}")`) so arguments are properly isolated and not subject to word-splitting or shell metacharacter interpretation
3. Changed `ARGS+=" --config ${INPUT_CONFIG}"` to `ARGS+=("--config" "${INPUT_CONFIG}")` so INPUT_CONFIG is a properly quoted, separate array element
4. Changed final command executions from `${COMMAND} ${ARGS}` to `"${COMMAND}" "${ARGS[@]}"` to use proper array expansion with double quotes, preventing shell metacharacters in any input from being interpreted as shell commands

