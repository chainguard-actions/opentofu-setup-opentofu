<!-- markdownlint-disable -->

# Hardening Report: opentofu--setup-opentofu/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opentofu--setup-opentofu/v2.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v6.0.3` — a version tag, not a full 40-character commit SHA. This means the action can be silently updated or replaced by a malicious commit without changing the workflow file. All 7 checkout steps in setup-tofu.yml are affected. The reference should be pinned to a full SHA, e.g. `actions/checkout@<40-hex-sha> # v6.0.3`.

Locations:

- `.github/workflows/setup-tofu.yml:26`
- `.github/workflows/setup-tofu.yml:40`
- `.github/workflows/setup-tofu.yml:60`
- `.github/workflows/setup-tofu.yml:85`
- `.github/workflows/setup-tofu.yml:110`
- `.github/workflows/setup-tofu.yml:127`
- `.github/workflows/setup-tofu.yml:156`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and no individual job defines its own `permissions:` block. Without explicit permissions, workflows run with the default repository token permissions, which may be overly broad (e.g. write access to contents, pull-requests, etc.). A minimal `permissions: {}` or specific scopes should be declared.

Locations:

- `.github/workflows/setup-tofu.yml:1`
- `.github/workflows/continuous-integration.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in setup-tofu.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing an attacker to inject arbitrary shell commands via matrix values or step outputs:

1. Line 50: `run: tofu version | grep ${{ matrix['tofu-versions']}}` — the matrix value is interpolated unquoted directly into the shell command.
2. Line 113: `run: echo "${{ steps.plan.outputs.stdout }}"` — step output is interpolated directly into the shell command; a malicious tofu plan output could inject shell commands.
3. Lines 130 & 134 (tofu-credentials-cloud job): `grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` — the env expression is interpolated directly into the run block.
4. Lines 159 & 163 (tofu-credentials-enterprise job): same pattern with `${{ env.TF_CLOUD_API_TOKEN }}`.

All `${{ ... }}` expressions must be moved to `env:` variables and then referenced as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/setup-tofu.yml:50`
- `.github/workflows/setup-tofu.yml:113`
- `.github/workflows/setup-tofu.yml:130`
- `.github/workflows/setup-tofu.yml:134`
- `.github/workflows/setup-tofu.yml:159`
- `.github/workflows/setup-tofu.yml:163`

### hardcoded-credentials (severity: high)

The workflow file `setup-tofu.yml` contains a hardcoded literal API token value assigned to `TF_CLOUD_API_TOKEN`: `'XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'`. Even though this appears to be a placeholder/test value, it is a non-expression literal assigned to a name containing `token`, matching the hardcoded-credentials pattern. Real credentials must never be stored as literals in workflow files; use `${{ secrets.TF_CLOUD_API_TOKEN }}` instead.

Locations:

- `.github/workflows/setup-tofu.yml:119`
- `.github/workflows/setup-tofu.yml:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all 4 findings in setup-tofu.yml and continuous-integration.yml:

1. unpinned-uses: Pinned all 7 `actions/checkout@v6.0.3` references to full SHA `df4cb1c069e1874edd31b4311f1884172cec0e10` with `# v6.0.3` comment.

2. missing-permissions: Added `permissions: {}` at top level of both workflow files. Added `permissions: contents: read` to each job in setup-tofu.yml (needed for checkout).

3. script-injection: Moved all `${{ }}` expressions out of `run:` blocks into `env:` blocks: matrix tofu-versions → TOFU_VERSION, steps.plan.outputs.stdout → PLAN_STDOUT, env.TF_CLOUD_API_TOKEN → TF_TOKEN (in both cloud and enterprise credential jobs).

4. hardcoded-credentials: Replaced hardcoded literal token strings with `${{ secrets.TF_CLOUD_API_TOKEN }}` in both tofu-credentials-cloud and tofu-credentials-enterprise jobs.

