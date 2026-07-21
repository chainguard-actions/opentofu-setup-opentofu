<!-- markdownlint-disable -->

# Hardening Report: opentofu--setup-opentofu/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opentofu--setup-opentofu/v1.0.7** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in setup-tofu.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing script injection. Affected lines:
- Line 51: `run: tofu version | grep ${{ matrix['tofu-versions']}}` — matrix value interpolated directly and unquoted into a shell command.
- Line 87: `run: echo "${{ steps.plan.outputs.stdout }}"` — step output interpolated directly into shell.
- Lines 101 & 104 (tofu-credentials-cloud job): `grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` — env context interpolated directly into shell commands.
- Lines 119 & 122 (tofu-credentials-enterprise job): same `grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` pattern.
All of these bypass shell quoting and allow an attacker-controlled value to execute arbitrary shell commands.

Locations:

- `.github/workflows/setup-tofu.yml:51`
- `.github/workflows/setup-tofu.yml:87`
- `.github/workflows/setup-tofu.yml:101`
- `.github/workflows/setup-tofu.yml:104`
- `.github/workflows/setup-tofu.yml:119`
- `.github/workflows/setup-tofu.yml:122`

### unpinned-uses (severity: high)

Several workflow files reference `actions/checkout@v4` using a mutable version tag instead of a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. Failing references:
- retag.yml: `uses: actions/checkout@v4`
- setup-tofu.yml: `uses: actions/checkout@v4` (appears 6 times across multiple jobs)
Fix: pin each reference to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/retag.yml:16`
- `.github/workflows/setup-tofu.yml:26`
- `.github/workflows/setup-tofu.yml:42`
- `.github/workflows/setup-tofu.yml:68`
- `.github/workflows/setup-tofu.yml:93`
- `.github/workflows/setup-tofu.yml:108`
- `.github/workflows/setup-tofu.yml:126`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job defines a `permissions:` block. Without explicit permissions, workflows run with the default token permissions (which may be `write-all` depending on repository settings), granting unnecessarily broad access. All three workflow files are affected:
- continuous-integration.yml
- retag.yml
- setup-tofu.yml
Fix: add a top-level `permissions: {}` (or minimal specific scopes) to each workflow file.

Locations:

- `.github/workflows/continuous-integration.yml:1`
- `.github/workflows/retag.yml:1`
- `.github/workflows/setup-tofu.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. script-injection (setup-tofu.yml): Moved all ${{ }} expressions from run: blocks into step-level env: blocks and referenced them as plain shell variables. Affected: matrix['tofu-versions'] grep (line 51), steps.plan.outputs.stdout echo (line 87), and four TF_CLOUD_API_TOKEN grep commands in the cloud and enterprise credential jobs (lines 101, 104, 119, 122).

2. unpinned-uses: Pinned all 7 occurrences of actions/checkout@v4 to the full SHA actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4 in retag.yml (1 occurrence) and setup-tofu.yml (6 occurrences).

3. missing-permissions: Added permissions: {} at the top level of continuous-integration.yml, retag.yml, and setup-tofu.yml. The retag.yml tag job also received permissions: contents: write since it pushes tags to the repository.

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced hardcoded literal token values ('XXXXXXXXXXXXXX.atlasv1.XXXX...') in TF_CLOUD_API_TOKEN environment variable with ${{ secrets.TF_CLOUD_API_TOKEN }} in both the tofu-credentials-cloud and tofu-credentials-enterprise jobs in .github/workflows/setup-tofu.yml. Credentials are now properly sourced from GitHub Secrets.

