<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.49.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.49.1** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Unquoted shell variable expansions of untrusted data in action/entrypoint.sh. INPUT_FILES (from inputs.files) is assigned to TARGET and used unquoted in `ls ${TARGET}` (line 17) and in ARGS which is executed unquoted as `${COMMAND} ${ARGS}` (lines 66-67). INPUT_CONFIG (from inputs.config) is appended to ARGS unquoted: `ARGS+=" --config ${INPUT_CONFIG}"` (line 63), then ARGS is executed unquoted. An attacker-controlled input value containing shell metacharacters (`;`, `|`, `&`, `$(...)`) can achieve command injection.

Locations:

- `action/entrypoint.sh:17`
- `action/entrypoint.sh:63`
- `action/entrypoint.sh:66`
- `action/entrypoint.sh:67`

### script-injection (severity: high)

Rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in post-release.yml. Affected steps include: (line 43) `run: echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV`; (line 49) `release-notes.py --tag ${{ env.TAG }}`; (line 54) `notes-file notes-${{ env.TAG }}.md`; (line ~80) `cargo build --target ${{ matrix.target }}`; (lines ~83-97) Build archive step with `${{ matrix.target }}`, `${{ env.BIN_NAME }}`, `${{ needs.create-release.outputs.tag }}`, `${{ matrix.os }}`, `${{ env.ASSET }}`; (line ~103) `tag="${{ needs.create-release.outputs.tag }}"`; (line ~104) `gh release upload "$tag" ${{ env.ASSET }}`; (line ~113) `gh release edit "${{ needs.create-release.outputs.tag }}"`

Locations:

- `.github/workflows/post-release.yml:43`
- `.github/workflows/post-release.yml:49`
- `.github/workflows/post-release.yml:54`
- `.github/workflows/post-release.yml:80`
- `.github/workflows/post-release.yml:83`
- `.github/workflows/post-release.yml:103`
- `.github/workflows/post-release.yml:104`
- `.github/workflows/post-release.yml:113`

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in template.yml. Affected steps: (line 36) `git config --global user.name '${{ github.actor }}'`; (line 39) `git remote add template ${{ env.TEMPLATE_URL }} && git fetch template ${{ env.TEMPLATE_BRANCH }}`; (line 41) `git merge template/${{ env.TEMPLATE_BRANCH }}`

Locations:

- `.github/workflows/template.yml:36`
- `.github/workflows/template.yml:39`
- `.github/workflows/template.yml:41`

### github-env-injection (severity: high)

A run: block writes a value derived from github.ref_name directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The offending line is: `run: echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV`. An attacker who can control the tag name (e.g. via a crafted tag push) could inject arbitrary environment variable definitions into subsequent steps.

Locations:

- `.github/workflows/post-release.yml:43`

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag or branch instead of a pinned 40-character SHA commit hash, making the action vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master, crate-ci/committed@master, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8, rickstaa/action-update-semver@v1, j178/prek-action@v2

Locations:

- `.github/workflows/audit.yml:30`
- `.github/workflows/ci.yml:42`
- `.github/workflows/committed.yml:24`
- `.github/workflows/maturin.yml:18`
- `.github/workflows/post-release.yml:38`
- `.github/workflows/pre-commit.yml:22`
- `.github/workflows/rust-next.yml:27`
- `.github/workflows/template.yml:31`
- `.github/workflows/test-action.yml:12`

### missing-permissions (severity: medium)

test-action.yml has no top-level `permissions:` key and no job-level `permissions:` block on any of its jobs (shallow, deep). Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad (e.g. write access to contents).

Locations:

- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 9 files:

1. entrypoint.sh (script-injection): Changed ARGS from a string to a bash array. Quoted TARGET in ls call. ARGS built with array syntax and expanded with "${ARGS[@]}" to prevent shell metacharacter injection from INPUT_FILES and INPUT_CONFIG.

2. post-release.yml (script-injection + github-env-injection + unpinned-uses): Moved all ${{ }} expressions from run: blocks to env: blocks. Fixed github-env-injection by sanitizing github.ref_name with printf/tr before writing to GITHUB_ENV. Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, and rickstaa/action-update-semver@v1 to full SHAs.

3. template.yml (script-injection + unpinned-uses): Moved ${{ github.actor }} to env: block as ACTOR. Replaced ${{ env.TEMPLATE_URL }} and ${{ env.TEMPLATE_BRANCH }} in run: blocks with plain $TEMPLATE_URL and $TEMPLATE_BRANCH env var references. Pinned actions/checkout@v6 to full SHA.

4. audit.yml (unpinned-uses): Pinned actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2 to full SHAs.

5. ci.yml (unpinned-uses): Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master to full SHAs.

6. committed.yml (unpinned-uses): Pinned actions/checkout@v6 and crate-ci/committed@master to full SHAs.

7. maturin.yml (unpinned-uses): Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8 to full SHAs.

8. pre-commit.yml (unpinned-uses): Pinned actions/checkout@v6 and j178/prek-action@v2 to full SHAs.

9. rust-next.yml (unpinned-uses): Pinned actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack to full SHAs.

10. test-action.yml (missing-permissions + unpinned-uses): Added top-level permissions: contents: read block. Pinned actions/checkout@v6 to full SHA.

### Iteration 2

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed two security findings: (1) github-env-injection in post-release.yml: sanitized the 'staging' variable (composed of RELEASE_TAG and MATRIX_TARGET) with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV; (2) script-injection in action/entrypoint.sh: added double-quotes around all three unquoted uses of `${_INSTALL_DIR}` (mkdir -p, unzip -d, tar -C) to prevent word-splitting on the runner.temp path.

