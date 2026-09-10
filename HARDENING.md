<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ inputs.matrix }}` is interpolated directly inside a `run:` heredoc block in the `prepare-matrix` step. GitHub Actions performs template substitution before the shell executes, so attacker-controlled content from `inputs.matrix` is written verbatim into the shell script. Although the heredoc delimiter is quoted (`<<'MATRIX_EXTENDED_INPUT_EOF'`), a value containing the heredoc terminator string can break out of the heredoc and execute arbitrary shell commands. The offending line is: `        ${{ inputs.matrix }}`

Locations:

- `action.yml:47`

### script-injection (severity: high)

Sub-rule (a): The expression `${{ steps.prepare-matrix.outputs.file }}` is interpolated directly inside a `run:` shell command in the `cleanup-matrix` step. `steps.*.outputs.*` is a workflow-controllable context value that flows through YAML template substitution before the shell sees it. A malicious value could inject shell metacharacters into the `rm -f` command. The offending line is: `      run: rm -f "${{ steps.prepare-matrix.outputs.file }}"`

Locations:

- `action.yml:111`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.matrix }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml:
1. (line 47) Moved `${{ inputs.matrix }}` from direct interpolation inside a heredoc to the step's `env:` block as `INPUT_MATRIX`. The shell now writes the value to the temp file using `printf '%s' "$INPUT_MATRIX" > "$matrix_file"`, eliminating any heredoc-escape or shell-injection risk.
2. (line 111) Moved `${{ steps.prepare-matrix.outputs.file }}` from direct interpolation in the `rm -f` command to the step's `env:` block as `MATRIX_FILE`, referenced safely as `"$MATRIX_FILE"` in the shell script.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/action.yml at the `prepare-matrix` step. The `$content` variable (derived from caller-controlled `inputs.matrix`) was being written directly to `$GITHUB_OUTPUT` without newline sanitization. Added `safe=$(printf '%s' "$content" | tr -d '\n\r')` and changed the echo to use `$safe` instead of `$content`, preventing an attacker from injecting arbitrary key=value pairs into the output file via embedded newlines.

