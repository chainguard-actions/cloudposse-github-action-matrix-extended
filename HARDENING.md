<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/0.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step uses `cloudposse/github-action-jq@v0`, which is pinned to a mutable tag (`v0`) rather than an immutable 40-character commit SHA. If the upstream repository is compromised or the tag is moved, this action will silently execute different code, enabling a supply-chain attack.

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned `cloudposse/github-action-jq@v0` to the immutable commit SHA `aff18a1f2e845b56fcfe995b01a558836dc2025b` in hardened/action/action.yml line 30. The original tag is preserved as a comment (`# v0`) for readability.

