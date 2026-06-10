<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.47.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.47.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansions of untrusted input data in action/entrypoint.sh. The env var TARGET (set from INPUT_FILES, which holds inputs.files — a caller-controlled value) is expanded unquoted in `ls ${TARGET}` (line 17), allowing shell metacharacter injection via word splitting and glob expansion. Additionally, ARGS (which accumulates TARGET and INPUT_CONFIG unquoted) is passed unquoted to the typos command on lines 59–60: `${COMMAND} ${ARGS} --format json | ...` and `${COMMAND} ${ARGS}`. An attacker-controlled value in inputs.files or inputs.config containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can cause arbitrary command execution.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:55`
- `action/entrypoint.sh:59`
- `action/entrypoint.sh:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh:
1. Quoted `${TARGET}` in the `ls` check on line 17: `ls "${TARGET}"` — prevents word splitting and glob expansion from caller-controlled INPUT_FILES.
2. Changed ARGS from an unquoted string variable to a bash array: `ARGS=("${TARGET}")` with `ARGS+=("--isolated")`, `ARGS+=("--write-changes")`, and `ARGS+=("--config" "${INPUT_CONFIG}")` for conditional flags. INPUT_CONFIG is now added as two separate array elements rather than concatenated into a string.
3. Changed command invocations from `${COMMAND} ${ARGS}` to `"${COMMAND}" "${ARGS[@]}"` — the quoted array expansion `"${ARGS[@]}"` ensures each argument is treated as a separate word with no shell metacharacter interpretation, preventing injection via inputs.files or inputs.config.

