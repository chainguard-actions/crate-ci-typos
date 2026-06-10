<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.47.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.47.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (b): Unquoted shell variable expansions of untrusted input data in action/entrypoint.sh. The env var TARGET (set from INPUT_FILES, which maps to inputs.files) is used unquoted in `ls ${TARGET}` and as the initial value of ARGS. INPUT_CONFIG (from inputs.config) is appended to ARGS unquoted: `ARGS+=" --config ${INPUT_CONFIG}"`. ARGS itself is then passed unquoted to the command invocations: `${COMMAND} ${ARGS}`. An attacker-controlled value in inputs.files or inputs.config containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) would be interpreted by the shell, enabling command injection.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:66`
- `action/entrypoint.sh:70`
- `action/entrypoint.sh:71`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in action/entrypoint.sh by: (1) quoting TARGET in the `ls "${TARGET}"` check on line 17; (2) converting the ARGS string variable to a bash array `ARGS=("${TARGET}")` so each argument is stored and expanded as a separate quoted word; (3) appending --config and INPUT_CONFIG as separate quoted array elements `ARGS+=("--config" "${INPUT_CONFIG}")` instead of unquoted string concatenation; (4) invoking the command with `"${COMMAND}" "${ARGS[@]}"` using proper array expansion. These changes prevent attacker-controlled values in inputs.files or inputs.config (containing shell metacharacters like `;`, `|`, `&`, `$(...)`) from being interpreted as shell commands.

