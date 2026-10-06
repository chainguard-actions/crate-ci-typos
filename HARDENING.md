<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.51.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.51.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) violation: Multiple unquoted shell variable expansions of untrusted inputs in action/entrypoint.sh. The env vars INPUT_FILES and INPUT_CONFIG are sourced from ${{ inputs.files }} and ${{ inputs.config }} respectively (workflow-controllable). They are used unquoted in shell commands, allowing shell metacharacter injection via word-splitting:
- Line 17: `ls ${TARGET}` — TARGET is set from INPUT_FILES (inputs.files), unquoted in command substitution
- Line 63: `ARGS+=" --config ${INPUT_CONFIG}"` — INPUT_CONFIG (inputs.config) appended unquoted to ARGS
- Lines 67-68: `${COMMAND} ${ARGS}` — ARGS contains user-controlled TARGET and INPUT_CONFIG values, unquoted, enabling argument injection and shell metacharacter exploitation

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:63`
- `action/entrypoint.sh:67`
- `action/entrypoint.sh:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in hardened/action/action/entrypoint.sh:
1. Quoted `${TARGET}` in `ls "${TARGET}"` (line 17) to prevent word-splitting from INPUT_FILES.
2. Converted ARGS from a string to a bash array: `ARGS=("${TARGET}")` so argument boundaries are preserved.
3. Changed `ARGS+=" --config ${INPUT_CONFIG}"` to `ARGS+=("--config" "${INPUT_CONFIG}")` so INPUT_CONFIG is properly quoted as a separate array element.
4. Changed `${COMMAND} ${ARGS}` to `"${COMMAND}" "${ARGS[@]}"` in both command invocations so the array is expanded with proper quoting, preventing word-splitting and shell metacharacter injection.

