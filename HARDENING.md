<!-- markdownlint-disable -->

# Hardening Report: cloudposse--github-action-matrix-extended/0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cloudposse--github-action-matrix-extended/0.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or branch names instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks.

- action.yml: `cloudposse/github-action-jq@v0`
- auto-readme.yml: `actions/checkout@v3`, `cloudposse/actions/github/create-pull-request@0.33.0`
- feature-branch.yml: `cloudposse/github-actions-workflows-github-action-composite/...@main`
- main-branch.yaml: `cloudposse/github-actions-workflows-github-action-composite/...@main`
- release.yml: `cloudposse/github-action-major-release-tagger@v1`
- test-negative.yml: `actions/checkout@v3`, `nick-fields/assert-action@v1`
- test-nestried-matrices-1-input-matrix.yml: `actions/checkout@v3`, `nick-fields/assert-action@v1`
- test-nestried-matrices-1.yml: `actions/checkout@v3`, `nick-fields/assert-action@v1`
- test-nestried-matrices-2.yml: `actions/checkout@v3`, `nick-fields/assert-action@v1`
- test-nestried-matrices-3.yml: `actions/checkout@v3`, `nick-fields/assert-action@v1`

Locations:

- `action.yml:28`
- `.github/workflows/auto-readme.yml:27`
- `.github/workflows/auto-readme.yml:52`
- `.github/workflows/feature-branch.yml:7`
- `.github/workflows/main-branch.yaml:7`
- `.github/workflows/release.yml:9`
- `.github/workflows/test-negative.yml:20`
- `.github/workflows/test-negative.yml:32`
- `.github/workflows/test-nestried-matrices-1-input-matrix.yml:20`
- `.github/workflows/test-nestried-matrices-1-input-matrix.yml:72`
- `.github/workflows/test-nestried-matrices-1.yml:20`
- `.github/workflows/test-nestried-matrices-1.yml:47`
- `.github/workflows/test-nestried-matrices-2.yml:20`
- `.github/workflows/test-nestried-matrices-2.yml:47`
- `.github/workflows/test-nestried-matrices-3.yml:20`
- `.github/workflows/test-nestried-matrices-3.yml:47`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks, meaning they run with the default (potentially broad) token permissions:
- auto-readme.yml
- feature-branch.yml
- release.yml
- test-negative.yml
- test-nestried-matrices-1-input-matrix.yml
- test-nestried-matrices-1.yml
- test-nestried-matrices-2.yml
- test-nestried-matrices-3.yml

Each should declare minimal explicit permissions (e.g. `permissions: read-all` or specific scopes).

Locations:

- `.github/workflows/auto-readme.yml:1`
- `.github/workflows/feature-branch.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/test-negative.yml:1`
- `.github/workflows/test-nestried-matrices-1-input-matrix.yml:1`
- `.github/workflows/test-nestried-matrices-1.yml:1`
- `.github/workflows/test-nestried-matrices-2.yml:1`
- `.github/workflows/test-nestried-matrices-3.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions

**Notes:**

Fixed all unpinned `uses:` references by resolving them to full 40-character commit SHAs using lookup_action_sha. Added top-level `permissions:` blocks to all 8 workflow files that were missing them. Specific changes:

**unpinned-uses fixes:**
- action.yml: cloudposse/github-action-jq@v0 → @aff18a1f2e845b56fcfe995b01a558836dc2025b
- auto-readme.yml: actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26; cloudposse/actions/github/create-pull-request@0.33.0 → @d4ad42a5434d60d082a2c4fd585c2f8acba4e719
- feature-branch.yml: composite workflow @main → @a4d0078b26faab74b046f3104f39a40f5436aa92
- main-branch.yaml: composite workflow @main → @a4d0078b26faab74b046f3104f39a40f5436aa92
- release.yml: cloudposse/github-action-major-release-tagger@v1 → @ad2b41fc978968182b88d938c6f51e662eca7700
- All 5 test workflows: actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26; nick-fields/assert-action@v1 → @1e012cc9f1bf73ccc96470b56c8887478c647e8a

**missing-permissions fixes:**
- auto-readme.yml: contents: write, pull-requests: write
- feature-branch.yml: contents: read, pull-requests: write
- release.yml: contents: write
- test-negative.yml, test-nestried-matrices-1.yml, test-nestried-matrices-2.yml, test-nestried-matrices-3.yml, test-nestried-matrices-1-input-matrix.yml: contents: read

