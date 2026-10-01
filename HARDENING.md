<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.matrix }}` is directly interpolated inside a `run:` shell script in the `prepare-matrix` step. Although it appears inside a single-quoted heredoc (`<<'MATRIX_EXTENDED_INPUT_EOF'`), GitHub Actions performs template substitution on the entire script text before the shell ever executes it — the single-quoted delimiter only prevents bash from expanding `$` variables, not the Actions runner from substituting `${{ inputs.matrix }}`. An attacker-controlled `matrix` input can inject arbitrary shell commands into the runner.

Locations:

- `action.yml:46`

### script-injection (severity: high)

Sub-rule (a): `${{ steps.prepare-matrix.outputs.file }}` is directly interpolated inside the `run:` shell command of the `cleanup-matrix` step: `rm -f "${{ steps.prepare-matrix.outputs.file }}". The `steps.*.outputs.*` context is a workflow-controllable value that is substituted into the script text before the shell executes it, enabling command injection (e.g. a path value containing shell metacharacters or newlines).

Locations:

- `action.yml:110`

### github-env-injection (severity: high)

The `prepare-matrix` step writes `$content` to `$GITHUB_OUTPUT` without sanitization: `echo "file=$content" >> "$GITHUB_OUTPUT"`. The variable `$content` is derived by reading the file that was populated with the untrusted `${{ inputs.matrix }}` input. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing a malicious `matrix` value containing newlines to inject arbitrary key=value pairs into the GitHub output environment.

Locations:

- `action.yml:51`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.matrix }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all four findings in action.yml:
1. Moved `${{ inputs.matrix }}` from the heredoc in prepare-matrix to an env: variable (MATRIX_INPUT), then wrote it to the temp file using `printf '%s' "$MATRIX_INPUT" > "$matrix_file"` — eliminating the script-injection via the heredoc.
2. Added `tr -d '\n\r'` sanitization before writing both `file=` values to $GITHUB_OUTPUT, preventing newline-based github-env-injection.
3. Moved `${{ steps.prepare-matrix.outputs.file }}` in the cleanup-matrix step to an env: variable (MATRIX_FILE) and referenced it as `"$MATRIX_FILE"` in the run: command, eliminating the script-injection there.

