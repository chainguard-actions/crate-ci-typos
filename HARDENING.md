<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.49.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.49.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings, violating rule (a). In post-release.yml: `echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV` (line 44), `${{ env.TAG }}` in release-notes command (line 50), `notes-${{ env.TAG }}.md` in gh release create (line 55), `cargo build --target ${{ matrix.target }}` (line ~98), build archive step with `${{ matrix.target }}`, `${{ env.BIN_NAME }}`, `${{ needs.create-release.outputs.tag }}`, `${{ matrix.os }}` (lines ~102-116), `tag="${{ needs.create-release.outputs.tag }}"` and `${{ env.ASSET }}` (line ~120), `gh release edit "${{ needs.create-release.outputs.tag }}"` (line ~128). In template.yml: `git config --global user.name '${{ github.actor }}'` (line 35), `git remote add template ${{ env.TEMPLATE_URL }} && git fetch template ${{ env.TEMPLATE_BRANCH }}` (line 38), `git merge template/${{ env.TEMPLATE_BRANCH }}` (line 40). Additionally, in action/entrypoint.sh, unquoted shell variable expansions of user-controlled inputs violate rule (b): `ls ${TARGET}` (line 17, TARGET from inputs.files), `ARGS+=" --config ${INPUT_CONFIG}"` (line 67, INPUT_CONFIG from inputs.config), and `${COMMAND} ${ARGS}` (lines 71-72, ARGS contains user-controlled values).

Locations:

- `.github/workflows/post-release.yml:44`
- `.github/workflows/post-release.yml:50`
- `.github/workflows/post-release.yml:55`
- `.github/workflows/post-release.yml:98`
- `.github/workflows/post-release.yml:103`
- `.github/workflows/post-release.yml:120`
- `.github/workflows/post-release.yml:128`
- `.github/workflows/template.yml:35`
- `.github/workflows/template.yml:38`
- `.github/workflows/template.yml:40`
- `action/entrypoint.sh:17`
- `action/entrypoint.sh:67`
- `action/entrypoint.sh:71`

### github-env-injection (severity: high)

In post-release.yml, the run: block writes a value derived from the github context directly to $GITHUB_ENV without sanitization: `echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV`. The value of github.ref_name is attacker-influenced (e.g., via a crafted tag name) and is written without the required `printf '%s' ... | tr -d '\n\r'` sanitization step, enabling newline injection into the environment file.

Locations:

- `.github/workflows/post-release.yml:44`

### unpinned-uses (severity: high)

All workflow files use mutable tag or branch refs instead of immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Unpinned references found include: actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master, crate-ci/committed@master, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8, rickstaa/action-update-semver@v1, j178/prek-action@v2.

Locations:

- `.github/workflows/audit.yml:30`
- `.github/workflows/ci.yml:44`
- `.github/workflows/committed.yml:22`
- `.github/workflows/maturin.yml:22`
- `.github/workflows/post-release.yml:38`
- `.github/workflows/pre-commit.yml:23`
- `.github/workflows/rust-next.yml:30`
- `.github/workflows/template.yml:30`
- `.github/workflows/test-action.yml:11`

### missing-permissions (severity: medium)

The workflow file test-action.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (shallow, deep). Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across multiple files:

1. script-injection: Moved all ${{ }} expressions from run: shell strings to env: blocks in post-release.yml and template.yml. In entrypoint.sh, quoted the TARGET variable in ls, converted ARGS from a string to a bash array, and properly quoted INPUT_CONFIG when appending to the array.

2. github-env-injection: In post-release.yml, the TAG step now sanitizes the REF_NAME value with `printf '%s' "$REF_NAME" | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. unpinned-uses: Pinned all 14 action references to full 40-character SHA commit hashes across audit.yml, ci.yml, committed.yml, maturin.yml, post-release.yml, pre-commit.yml, rust-next.yml, template.yml, and test-action.yml.

4. missing-permissions: Added `permissions: contents: read` to test-action.yml.

