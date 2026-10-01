<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/v0.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/v0.2.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ }}` expressions into shell commands, violating rule (a).

1. Line 46 (step `prepare-matrix`): `${{ inputs.matrix }}` is interpolated directly inside a heredoc within a `run:` block. Although the heredoc delimiter is quoted (`'MATRIX_EXTENDED_INPUT_EOF'`), the GitHub Actions template substitution of `${{ inputs.matrix }}` occurs before the shell executes, so attacker-controlled content is written verbatim into the running script — enabling shell metacharacter injection.

2. Line 108 (step `cleanup-matrix`): `${{ steps.prepare-matrix.outputs.file }}` is interpolated directly inside a `run:` shell command (`rm -f "${{ steps.prepare-matrix.outputs.file }}"`). A step output derived from attacker-controlled input is injected into the shell command string without quoting or sanitization.

Locations:

- `action.yml:46`
- `action.yml:108`

### github-env-injection (severity: high)

The `prepare-matrix` run block (step id: `prepare-matrix`) writes values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. Line 51: `echo "file=$content" >> "$GITHUB_OUTPUT"` — `$content` is read from a temp file whose contents were written from `${{ inputs.matrix }}` (attacker-controlled). A newline-containing value can inject additional key=value pairs into `$GITHUB_OUTPUT`.

2. Line 54: `echo "file=$matrix_file" >> "$GITHUB_OUTPUT"` — `$matrix_file` is constructed using `${RUNNER_TEMP}` (set via `env:` from `${{ runner.temp }}`). Although `runner.temp` is typically safe, it is a workflow-context value written to `$GITHUB_OUTPUT` without sanitization, violating the check's requirement for sanitization before every write when the source is not a literal computed in the same block.

Locations:

- `action.yml:51`
- `action.yml:54`

### unpinned-uses (severity: high)

The composite action step `matrix` references `cloudposse/github-action-jq@v0` using a mutable version tag (`@v0`) instead of a pinned 40-character commit SHA. A mutable tag can be silently updated to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `cloudposse/github-action-jq@<40-char-sha> # v0`.

Locations:

- `action.yml:57`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.matrix }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all four findings in action.yml:
1. script-injection (lines 46/47): Moved `${{ inputs.matrix }}` from a heredoc in the run block to an env var `MATRIX_INPUT`, and rewrote the file-writing to use `printf '%s' "$MATRIX_INPUT" > "$matrix_file"` — no template expression in the shell script.
2. script-injection (line 108): Moved `${{ steps.prepare-matrix.outputs.file }}` from the `rm -f` command to an env var `MATRIX_FILE` in the cleanup-matrix step's `env:` block.
3. github-env-injection (lines 51/54): Added sanitization with `printf '%s' ... | tr -d '\n\r'` before writing both `$content` and `$matrix_file` to `$GITHUB_OUTPUT`.
4. unpinned-uses (line 57): Pinned `cloudposse/github-action-jq@v0` to full commit SHA `aff18a1f2e845b56fcfe995b01a558836dc2025b` with `# v0` comment.

