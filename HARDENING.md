<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.50.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.50.1** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansions of untrusted data in action/entrypoint.sh. INPUT_FILES (sourced from inputs.files via env var) is assigned to TARGET without quotes and then used unquoted in `ls ${TARGET}` (line 17). ARGS is built from TARGET (line 53) and INPUT_CONFIG is appended unquoted (`ARGS+=" --config ${INPUT_CONFIG}"`, line 67), then the entire ARGS string is passed unquoted to the command at lines 71-72 (`${COMMAND} ${ARGS}`). An attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, etc.) can achieve command injection.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:67`
- `action/entrypoint.sh:71`
- `action/entrypoint.sh:72`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions interpolated directly inside run: shell commands in post-release.yml. Affected lines include: line 45 (`echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV`), line 51 (`./.github/workflows/release-notes.py --tag ${{ env.TAG }} ...`), line 56 (`gh release create ... --notes-file notes-${{ env.TAG }}.md`), line 102 (`cargo build --target ${{ matrix.target }} ...`), and the Build archive block (~lines 105-115) which uses `${{ matrix.target }}`, `${{ env.BIN_NAME }}`, `${{ needs.create-release.outputs.tag }}`, and `${{ matrix.os }}` directly in shell commands. The Upload release archive step also uses `tag="${{ needs.create-release.outputs.tag }}"` and `${{ env.ASSET }}` directly in a run: block.

Locations:

- `.github/workflows/post-release.yml:45`
- `.github/workflows/post-release.yml:51`
- `.github/workflows/post-release.yml:56`
- `.github/workflows/post-release.yml:102`
- `.github/workflows/post-release.yml:106`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions interpolated directly inside run: shell commands in template.yml. Line 36: `git config --global user.name '${{ github.actor }}'` — github.actor is attacker-controlled and injected directly into a shell command. Line 39: `git remote add template ${{ env.TEMPLATE_URL }} && git fetch template ${{ env.TEMPLATE_BRANCH }}` — env vars injected directly into shell. Line 41: `git checkout -b template-update && git merge template/${{ env.TEMPLATE_BRANCH }} ...` — env.TEMPLATE_BRANCH injected directly into shell.

Locations:

- `.github/workflows/template.yml:36`
- `.github/workflows/template.yml:39`
- `.github/workflows/template.yml:41`

### github-env-injection (severity: high)

Unsanitized values written to $GITHUB_ENV in post-release.yml. Line 45: `echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV` writes github.ref_name (attacker-controllable via tag push) directly to GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization. In the Build archive run block (~lines 105-120), the variable `$staging` is constructed from `${{ needs.create-release.outputs.tag }}` and `${{ matrix.target }}` (both workflow-controlled) and then written to GITHUB_ENV via `echo "ASSET=$staging.zip" >> $GITHUB_ENV` and `echo "ASSET=$staging.tar.gz" >> $GITHUB_ENV` without sanitization, enabling newline injection attacks.

Locations:

- `.github/workflows/post-release.yml:45`
- `.github/workflows/post-release.yml:115`
- `.github/workflows/post-release.yml:119`

### unpinned-uses (severity: high)

Multiple workflow files use unpinned action references (tags or branch names instead of full 40-character SHA commit hashes), making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Failing references include: actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master, crate-ci/committed@master, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8, rickstaa/action-update-semver@v1, j178/prek-action@v2.

Locations:

- `.github/workflows/audit.yml:30`
- `.github/workflows/audit.yml:32`
- `.github/workflows/ci.yml:40`
- `.github/workflows/ci.yml:42`
- `.github/workflows/ci.yml:155`
- `.github/workflows/committed.yml:22`
- `.github/workflows/committed.yml:24`
- `.github/workflows/maturin.yml:20`
- `.github/workflows/post-release.yml:40`
- `.github/workflows/pre-commit.yml:22`
- `.github/workflows/pre-commit.yml:23`
- `.github/workflows/rust-next.yml:30`
- `.github/workflows/template.yml:31`
- `.github/workflows/test-action.yml:12`

### missing-permissions (severity: medium)

test-action.yml has no top-level `permissions:` key and neither of its jobs (shallow, deep) defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents by default for many repositories).

Locations:

- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 8 files:

1. entrypoint.sh: Converted ARGS string to bash array, quoted ls "${TARGET}", used array syntax for --config flag, invoked command as "${COMMAND}" "${ARGS[@]}".

2. post-release.yml: Moved all ${{ }} expressions from run: shell commands into env: blocks (MATRIX_TARGET, MATRIX_OS, RELEASE_TAG, REF_NAME). Sanitized GITHUB_ENV writes with printf '%s' | tr -d '\n\r'. Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, rickstaa/action-update-semver@v1.

3. template.yml: Moved ${{ github.actor }}, ${{ env.TEMPLATE_URL }}, ${{ env.TEMPLATE_BRANCH }} into env: blocks. Pinned actions/checkout@v6.

4. audit.yml: Pinned actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2.

5. ci.yml: Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master.

6. committed.yml: Pinned actions/checkout@v6, crate-ci/committed@master.

7. maturin.yml: Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8.

8. pre-commit.yml: Pinned actions/checkout@v6, j178/prek-action@v2.

9. rust-next.yml: Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack.

10. test-action.yml: Added top-level permissions: contents: read. Pinned actions/checkout@v6.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions of `${_INSTALL_DIR}` in hardened/action/action/entrypoint.sh (lines 43, 45, 47). Changed `mkdir -p ${_INSTALL_DIR}`, `unzip -o "${FILE_NAME}" -d ${_INSTALL_DIR} ${CMD_NAME}.exe`, and `tar -xzvf "${FILE_NAME}" -C ${_INSTALL_DIR} ./${CMD_NAME}` to use `"${_INSTALL_DIR}"` with double quotes in all three cases. This prevents shell metacharacter interpretation of the value derived from `${{ runner.temp }}` via the INSTALL_DIR environment variable.

