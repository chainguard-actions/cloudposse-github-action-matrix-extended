<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/v0.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/v0.2.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In the `prepare-matrix` step, `${{ inputs.matrix }}` is directly interpolated into the `run:` shell script. Although it appears inside a single-quoted heredoc delimiter (`<<'MATRIX_EXTENDED_INPUT_EOF'`), GitHub Actions template substitution occurs before the shell ever sees the script, so the raw value of `inputs.matrix` is injected into the script text. An attacker-controlled matrix value containing shell metacharacters or newlines can break out of the heredoc and execute arbitrary commands.

Locations:

- `action.yml:47`

### script-injection (severity: high)

Sub-rule (a): In the `cleanup-matrix` step, `${{ steps.prepare-matrix.outputs.file }}` is directly interpolated into the `run:` shell command `rm -f "${{ steps.prepare-matrix.outputs.file }}". The step output value is substituted into the shell command string by the Actions runner before the shell executes it, allowing a crafted output value to inject arbitrary shell commands.

Locations:

- `action.yml:96`

### unpinned-uses (severity: high)

The following `uses:` references are pinned to mutable tags or branch names rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- `action.yml`: `cloudposse/github-action-jq@v0` (tag `v0`)
- `.github/workflows/branch.yml`: `cloudposse/.github/.github/workflows/shared-github-action.yml@main` (branch `main`)
- `.github/workflows/release.yml`: `cloudposse/.github/.github/workflows/shared-release-branches.yml@main` (branch `main`)

Locations:

- `action.yml:55`
- `.github/workflows/branch.yml:23`
- `.github/workflows/release.yml:12`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any job. Without explicit permissions, workflows inherit the default repository token permissions (which may be `write-all` depending on repository settings), violating the principle of least privilege.

Locations:

- `.github/workflows/test-negative.yml:1`
- `.github/workflows/test-nestried-matrices-1.yml:1`
- `.github/workflows/test-nestried-matrices-2.yml:1`
- `.github/workflows/test-nestried-matrices-3.yml:1`
- `.github/workflows/test-nestried-matrices-1-input-matrix.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.matrix }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings:
1. script-injection (prepare-matrix step): Moved `${{ inputs.matrix }}` from the heredoc in the run: block to the env: block as INPUT_MATRIX, then used `printf '%s' "$INPUT_MATRIX" > "$matrix_file"` to write it safely.
2. script-injection (cleanup-matrix step): Moved `${{ steps.prepare-matrix.outputs.file }}` from the rm command to the env: block as MATRIX_FILE, referenced as `"$MATRIX_FILE"`.
3. Pinned cloudposse/github-action-jq@v0 → @aff18a1f2e845b56fcfe995b01a558836dc2025b # v0 in action.yml.
4. Pinned cloudposse/.github shared-github-action.yml@main → @4e05ff6c113efa9322288cedbc7c8950c22616cc # main in branch.yml.
5. Pinned cloudposse/.github shared-release-branches.yml@main → @4e05ff6c113efa9322288cedbc7c8950c22616cc # main in release.yml.
6. Added `permissions: {}` to all 5 test workflow files that lacked permissions blocks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in action.yml at line 54. In the prepare-matrix step's if-branch (when inputs.matrix is a file path), the $content variable was written directly to $GITHUB_OUTPUT without newline sanitization. Added `safe_content="$(printf '%s' "$content" | tr -d '\n\r')"` and changed the echo to use `$safe_content` instead of `$content`. This prevents an attacker from injecting additional key=value pairs into $GITHUB_OUTPUT by embedding newlines in the inputs.matrix value. The else-branch ($matrix_file) was already safe as it's constructed from controlled values only.

