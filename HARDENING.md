<!-- markdownlint-disable -->

# Hardening Report: crate-ci--typos/v1.47.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **crate-ci--typos/v1.47.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag or branch refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced action is compromised or its tag is moved.

audit.yml: actions/checkout@v6, actions-rs/audit-check@v1, EmbarkStudios/cargo-deny-action@v2
ci.yml: actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack, github/codeql-action/upload-sarif@v4, coverallsapp/github-action@master
committed.yml: actions/checkout@v6, crate-ci/committed@master
maturin.yml: actions/checkout@v6, dtolnay/rust-toolchain@stable, PyO3/maturin-action@v1, actions/upload-artifact@v7, actions/download-artifact@v8
post-release.yml: actions/checkout@v6, dtolnay/rust-toolchain@stable, rickstaa/action-update-semver@v1
pre-commit.yml: actions/checkout@v6, j178/prek-action@v2
rust-next.yml: actions/checkout@v6, dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, taiki-e/install-action@cargo-hack
template.yml: actions/checkout@v6
test-action.yml: actions/checkout@v6

Locations:

- `.github/workflows/audit.yml:29`
- `.github/workflows/ci.yml:37`
- `.github/workflows/committed.yml:21`
- `.github/workflows/maturin.yml:17`
- `.github/workflows/post-release.yml:39`
- `.github/workflows/pre-commit.yml:27`
- `.github/workflows/rust-next.yml:29`
- `.github/workflows/template.yml:30`
- `.github/workflows/test-action.yml:10`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions (sub-rule a), allowing YAML template substitution to inject shell metacharacters before the shell parses the command.

post-release.yml:
- 'run: echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV' — github.ref_name interpolated directly in run:
- 'release-notes.py --tag ${{ env.TAG }} --output notes-${{ env.TAG }}.md' — env.TAG interpolated directly
- 'run: gh release create $TAG ... --notes-file notes-${{ env.TAG }}.md' — env.TAG interpolated directly
- 'run: cargo build --target ${{ matrix.target }} --verbose --release' — matrix.target interpolated directly
- Build archive block: outdir, staging, matrix.os, matrix.target, env.BIN_NAME, needs.create-release.outputs.tag all interpolated directly in run:
- 'tag="${{ needs.create-release.outputs.tag }}"' and 'gh release upload "$tag" ${{ env.ASSET }}' — interpolated directly
- 'run: gh release edit "${{ needs.create-release.outputs.tag }}" --draft=false' — interpolated directly

template.yml:
- 'git config --global user.name \'${{ github.actor }}\'' — github.actor interpolated directly
- 'git remote add template ${{ env.TEMPLATE_URL }} && git fetch template ${{ env.TEMPLATE_BRANCH }}' — env.* interpolated directly
- 'git merge template/${{ env.TEMPLATE_BRANCH }}' — env.TEMPLATE_BRANCH interpolated directly

Locations:

- `.github/workflows/post-release.yml:44`
- `.github/workflows/post-release.yml:50`
- `.github/workflows/post-release.yml:55`
- `.github/workflows/post-release.yml:96`
- `.github/workflows/post-release.yml:100`
- `.github/workflows/post-release.yml:120`
- `.github/workflows/post-release.yml:131`
- `.github/workflows/template.yml:35`
- `.github/workflows/template.yml:38`
- `.github/workflows/template.yml:40`

### github-env-injection (severity: high)

run: blocks write values derived from workflow-controlled contexts to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'run: echo "TAG=${{ github.ref_name }}" >> $GITHUB_ENV' — github.ref_name (attacker-controllable via tag push) is written directly to GITHUB_ENV.

2. In the 'Build archive' step, the variable $staging is constructed from ${{ matrix.target }}, ${{ needs.create-release.outputs.tag }}, and ${{ env.BIN_NAME }} via direct ${{ }} interpolation, then written to GITHUB_ENV via 'echo "ASSET=$staging.zip" >> $GITHUB_ENV' and 'echo "ASSET=$staging.tar.gz" >> $GITHUB_ENV' without sanitization.

Locations:

- `.github/workflows/post-release.yml:44`
- `.github/workflows/post-release.yml:110`
- `.github/workflows/post-release.yml:113`

### missing-permissions (severity: medium)

test-action.yml has no top-level permissions: key and none of its jobs (shallow, deep) define a job-level permissions: block. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad (e.g., write access to contents).

Locations:

- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 9 workflow files:

1. unpinned-uses: Pinned all 14 unique action references to their full 40-char SHA hashes with original tag/ref preserved as comments. Files updated: audit.yml, ci.yml, committed.yml, maturin.yml, post-release.yml, pre-commit.yml, rust-next.yml, template.yml, test-action.yml.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks in post-release.yml (github.ref_name→REF_NAME, matrix.target→MATRIX_TARGET, matrix.os→MATRIX_OS, needs.create-release.outputs.tag→RELEASE_TAG) and template.yml (github.actor→ACTOR, env.TEMPLATE_URL→TMPL_URL, env.TEMPLATE_BRANCH→TMPL_BRANCH). Shell scripts now reference plain env vars.

3. github-env-injection: Sanitized values written to $GITHUB_ENV using `printf '%s' "$VAR" | tr -d '\n\r'` before writing. Applied to TAG (from github.ref_name) and ASSET (from staging path) in post-release.yml.

4. missing-permissions: Added `permissions: contents: read` top-level block to test-action.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted shell variable expansions in hardened/action/action/entrypoint.sh:
1. Line 17: Added double-quotes around `${TARGET}` in the `ls` call: `ls "${TARGET}"`.
2. Lines 51-59: Converted the `ARGS` variable from a plain string to a bash array. `ARGS` is now initialized as `ARGS=("${TARGET}")`, flags are appended with `ARGS+=("--flag")`, and `INPUT_CONFIG` is appended as two separate elements `ARGS+=("--config" "${INPUT_CONFIG}")`. The command is invoked as `"${COMMAND}" "${ARGS[@]}"` which preserves argument boundaries and prevents word splitting and glob expansion of user-controlled data.

