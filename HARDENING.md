<!-- markdownlint-disable -->

# Hardening Report: opentofu--setup-opentofu/v1.0.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opentofu--setup-opentofu/v1.0.8** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in setup-tofu.yml directly interpolate `${{ }}` expressions into shell commands, enabling script injection. (1) `run: tofu version | grep ${{ matrix['tofu-versions']}}` — matrix context injected directly into a shell grep argument with no quoting. (2) `run: echo "${{ steps.plan.outputs.stdout }}"` — step output injected directly into echo. (3) Four `run:` blocks in the tofu-credentials-cloud and tofu-credentials-enterprise jobs contain `grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` — env context injected directly into shell. Any of these values could contain shell metacharacters that execute arbitrary commands.

Locations:

- `.github/workflows/setup-tofu.yml:59`
- `.github/workflows/setup-tofu.yml:125`
- `.github/workflows/setup-tofu.yml:147`
- `.github/workflows/setup-tofu.yml:153`
- `.github/workflows/setup-tofu.yml:173`
- `.github/workflows/setup-tofu.yml:179`

### unpinned-uses (severity: high)

Multiple workflow files reference `actions/checkout@v4` using a mutable tag instead of a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk. Affected references: `actions/checkout@v4` in retag.yml (line 15) and in setup-tofu.yml (lines 26, 46, 69, 106, 136, 161, 186).

Locations:

- `.github/workflows/retag.yml:15`
- `.github/workflows/setup-tofu.yml:26`
- `.github/workflows/setup-tofu.yml:46`
- `.github/workflows/setup-tofu.yml:69`
- `.github/workflows/setup-tofu.yml:106`
- `.github/workflows/setup-tofu.yml:136`
- `.github/workflows/setup-tofu.yml:161`
- `.github/workflows/setup-tofu.yml:186`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of their jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the default (potentially broad) token permissions. All three files — continuous-integration.yml, retag.yml, and setup-tofu.yml — are affected. The retag.yml job pushes tags and needs only `contents: write`; all others should be scoped to the minimum required permissions.

Locations:

- `.github/workflows/continuous-integration.yml:1`
- `.github/workflows/retag.yml:1`
- `.github/workflows/setup-tofu.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. script-injection (6 locations in setup-tofu.yml): Moved all ${{ }} expressions out of run: shell strings into step-level env: blocks, then referenced them as plain shell variables ($TOFU_VERSION, $PLAN_STDOUT, $TF_TOKEN).

2. unpinned-uses (8 locations): Pinned all actions/checkout@v4 references to the full commit SHA actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4 in both retag.yml and setup-tofu.yml.

3. missing-permissions (3 files): Added top-level permissions blocks — contents: read for continuous-integration.yml and setup-tofu.yml; contents: write for retag.yml (which pushes tags).

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal token value 'XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXX...' with ${{ secrets.TF_CLOUD_API_TOKEN }} in two jobs within .github/workflows/setup-tofu.yml: (1) tofu-credentials-cloud job env block (previously line 100), and (2) tofu-credentials-enterprise job env block (previously line 133). The downstream steps that reference ${{ env.TF_CLOUD_API_TOKEN }} continue to work correctly since they now resolve to the secret value at runtime.

