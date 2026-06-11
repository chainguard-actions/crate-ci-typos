<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.44.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **crate-ci--typos/v1.44.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, env vars holding user-controlled input values are expanded unquoted in shell commands, allowing shell metacharacter injection. Specifically: (1) `ls ${TARGET}` where TARGET comes from INPUT_FILES (user input via inputs.files); (2) `ARGS+=" --config ${INPUT_CONFIG}"` where INPUT_CONFIG is unquoted; (3) `${COMMAND} ${ARGS}` where ARGS contains user-controlled data (TARGET and INPUT_CONFIG). An attacker could supply a value like `"; malicious-command #` via the inputs to inject arbitrary shell commands.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:57`
- `action/entrypoint.sh:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection vulnerabilities in action/entrypoint.sh: (1) Quoted `${TARGET}` in the `ls` call on line 17 to prevent shell metacharacter injection from INPUT_FILES. (2) Converted ARGS from a plain string to a bash array — each argument (TARGET, --isolated, --write-changes, --config, INPUT_CONFIG) is now added as a separate quoted array element, preventing word-splitting and metacharacter injection. (3) The final command invocations now use `"${COMMAND}" "${ARGS[@]}"` which safely expands the array preserving argument boundaries, preventing injection via user-controlled INPUT_FILES or INPUT_CONFIG values.

