<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.50.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.50.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansions of untrusted input values in action/entrypoint.sh. INPUT_FILES (from inputs.files) is assigned to TARGET without quoting, then used unquoted in `ls ${TARGET}` (line 17) and `ARGS="${TARGET}"` (line 46). INPUT_CONFIG (from inputs.config) is used unquoted inside `ARGS+=" --config ${INPUT_CONFIG}"` (line 52). The accumulated ARGS variable is then passed unquoted to the command on lines 55-56: `${COMMAND} ${ARGS}`. An attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) can break out of the intended command context.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:46`
- `action/entrypoint.sh:52`
- `action/entrypoint.sh:55`
- `action/entrypoint.sh:56`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks in post-release.yml. Multiple expressions are interpolated directly into shell commands before the shell ever sees them: `echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV` (line 43), `notes-${{ env.TAG }}.md` in a gh release create command (line 51), `cargo build --target ${{ matrix.target }}` (line 79), `outdir="./target/${{ matrix.target }}/release"` and `staging="${{ env.BIN_NAME }}-${{ needs.create-release.outputs.tag }}-${{ matrix.target }}"` in the Build archive step (lines 87-88), `if [ "${{ matrix.os }}" = "windows-2022" ]` (line 93), `cp "target/${{ matrix.target }}/release/${{ env.BIN_NAME }}.exe"` (line 94), `gh release upload "$tag" ${{ env.ASSET }}` (line 110), and `gh release edit "${{ needs.create-release.outputs.tag }}" --draft=false` (line 120). Any of these context values flowing through YAML template substitution can inject shell metacharacters.

Locations:

- `.github/workflows/post-release.yml:43`
- `.github/workflows/post-release.yml:51`
- `.github/workflows/post-release.yml:79`
- `.github/workflows/post-release.yml:87`
- `.github/workflows/post-release.yml:93`
- `.github/workflows/post-release.yml:110`
- `.github/workflows/post-release.yml:120`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks in template.yml. `git config --global user.name '${{ github.actor }}'` (line 33) interpolates the GitHub actor name directly into a shell command — an attacker with a crafted username can inject shell commands. `git remote add template ${{ env.TEMPLATE_URL }} && git fetch template ${{ env.TEMPLATE_BRANCH }}` (line 35) and `git merge template/${{ env.TEMPLATE_BRANCH }}` (line 37) interpolate env context values directly into shell commands.

Locations:

- `.github/workflows/template.yml:33`
- `.github/workflows/template.yml:35`
- `.github/workflows/template.yml:37`

### github-env-injection (severity: high)

Unsanitized writes to $GITHUB_ENV in post-release.yml. (1) `echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV` (line 43) writes the git ref name — which can contain newlines — directly to GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A crafted tag name could inject additional environment variables. (2) In the Build archive step, `echo "ASSET=$staging.zip" >> $GITHUB_ENV` and `echo "ASSET=$staging.tar.gz" >> $GITHUB_ENV` write a value derived from `${{ matrix.target }}` and `${{ needs.create-release.outputs.tag }}` (both untrusted) to GITHUB_ENV without sanitization.

Locations:

- `.github/workflows/post-release.yml:43`
- `.github/workflows/post-release.yml:99`
- `.github/workflows/post-release.yml:103`

### missing-permissions (severity: medium)

The workflow file test-action.yml has no top-level `permissions:` key and none of its jobs (shallow, deep) define job-level `permissions:` blocks. This means the workflow runs with the default GitHub token permissions, which may be broader than necessary (e.g., write access to contents on push events).

Locations:

- `.github/workflows/test-action.yml:1`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or branch names instead of immutable 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved to a malicious commit. Affected references include: actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master (ci.yml); actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2 (audit.yml); actions/checkout@v6, crate-ci/committed@master (committed.yml); actions/checkout@v6, dtolnay/rust-toolchain@stable, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8 (maturin.yml); actions/checkout@v6, dtolnay/rust-toolchain@stable, rickstaa/action-update-semver@v1 (post-release.yml); actions/checkout@v6, j178/prek-action@v2 (pre-commit.yml); actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack (rust-next.yml); actions/checkout@v6 (template.yml); actions/checkout@v6 (test-action.yml).

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/audit.yml:1`
- `.github/workflows/committed.yml:1`
- `.github/workflows/maturin.yml:1`
- `.github/workflows/post-release.yml:1`
- `.github/workflows/pre-commit.yml:1`
- `.github/workflows/rust-next.yml:1`
- `.github/workflows/template.yml:1`
- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all findings across 10 files:

1. action/entrypoint.sh: Changed ARGS from a string to a bash array to prevent word splitting and glob expansion. Quoted TARGET in ls check. INPUT_CONFIG added as separate array elements. Command executed with "${COMMAND}" "${ARGS[@]}".

2. .github/workflows/post-release.yml: Moved all ${{ }} expressions from run: blocks to env: blocks (REF_NAME, MATRIX_TARGET, MATRIX_OS, RELEASE_TAG). Sanitized all GITHUB_ENV writes with printf '%s' | tr -d '\n\r'. Pinned actions/checkout, dtolnay/rust-toolchain, rickstaa/action-update-semver to full SHAs.

3. .github/workflows/template.yml: Moved ${{ github.actor }} to env: block as ACTOR. Used $TEMPLATE_URL and $TEMPLATE_BRANCH (already env vars) directly in shell. Pinned actions/checkout to full SHA.

4. .github/workflows/test-action.yml: Added top-level permissions: contents: read. Pinned actions/checkout to full SHA.

5. .github/workflows/ci.yml: Pinned all 6 action references (actions/checkout, dtolnay/rust-toolchain, Swatinem/rust-cache, taiki-e/install-action, github/codeql-action/upload-sarif, coverallsapp/github-action) to full SHAs.

6. .github/workflows/audit.yml: Pinned actions/checkout, actions-rs/audit-check, EmbarkStudios/cargo-deny-action to full SHAs.

7. .github/workflows/committed.yml: Pinned actions/checkout, crate-ci/committed to full SHAs.

8. .github/workflows/maturin.yml: Pinned actions/checkout, dtolnay/rust-toolchain, PyO3/maturin-action, actions/upload-artifact, actions/download-artifact to full SHAs.

9. .github/workflows/pre-commit.yml: Pinned actions/checkout, j178/prek-action to full SHAs.

10. .github/workflows/rust-next.yml: Pinned actions/checkout, dtolnay/rust-toolchain, Swatinem/rust-cache, taiki-e/install-action to full SHAs.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted expansions of `${_INSTALL_DIR}` in `hardened/action/action/entrypoint.sh`: `mkdir -p ${_INSTALL_DIR}` → `mkdir -p "${_INSTALL_DIR}"`, `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR} ${CMD_NAME}.exe` → `unzip -o "${FILE_NAME}" -d "${_INSTALL_DIR}" ${CMD_NAME}.exe`, and `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR} ./${CMD_NAME}` → `tar -xzvf "${FILE_NAME}" -C "${_INSTALL_DIR}" ./${CMD_NAME}`. The variable `_INSTALL_DIR` is derived from the `INSTALL_DIR` env var which is set from `${{ runner.temp }}` in action.yml. Proper double-quoting ensures that any spaces or special characters in the runner temp path are handled safely.

