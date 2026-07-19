<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.48.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.48.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the env var INPUT_FILES (sourced from inputs.files) is assigned to TARGET without quoting and then used unquoted in `ls ${TARGET}` and as `ARGS="${TARGET}"`. The final command `${COMMAND} ${ARGS}` is also executed unquoted, meaning shell metacharacters in the input value are interpreted by the shell. An attacker-controlled `inputs.files` value could inject arbitrary shell commands.

Locations:

- `action/entrypoint.sh:16`
- `action/entrypoint.sh:17`
- `action/entrypoint.sh:50`

### script-injection (severity: high)

Rule (b) violation: In action/entrypoint.sh, the env var INPUT_CONFIG (sourced from inputs.config) is appended to ARGS unquoted: `ARGS+=" --config ${INPUT_CONFIG}"`. The final command `${COMMAND} ${ARGS}` is executed unquoted, so shell metacharacters in INPUT_CONFIG are interpreted by the shell. An attacker-controlled `inputs.config` value could inject arbitrary shell commands.

Locations:

- `action/entrypoint.sh:58`
- `action/entrypoint.sh:61`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action/entrypoint.sh:
1. Quoted TARGET in the `ls` call: `ls "${TARGET}"` to prevent shell metacharacter interpretation from INPUT_FILES.
2. Converted ARGS from a plain string to a bash array: `ARGS=("${TARGET}")` so each argument is a separate, properly-quoted element.
3. Appended --config and INPUT_CONFIG as separate quoted array elements: `ARGS+=("--config" "${INPUT_CONFIG}")` instead of unquoted string concatenation.
4. Executed the final commands using quoted array expansion: `"${COMMAND}" "${ARGS[@]}"` to prevent word splitting and glob expansion of attacker-controlled values.

