<!-- markdownlint-disable -->

# Hardening Report: opentofu--setup-opentofu/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opentofu--setup-opentofu/v1.0.6** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct GitHub Actions expression interpolation (${{ ... }}) inside run: shell commands — sub-rule (a) violations. (1) Line 49: `run: tofu version | grep ${{ matrix['tofu-versions']}}` — matrix value interpolated directly into shell. (2) Line 109: `run: echo "${{ steps.plan.outputs.stdout }}"` — step output interpolated directly into shell. (3) Lines 122 and 127: `cat ... | grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` in tofu-credentials-cloud job. (4) Lines 149 and 154: same pattern in tofu-credentials-enterprise job. All bypass shell quoting and allow expression values to be parsed as shell metacharacters.

Locations:

- `.github/workflows/setup-tofu.yml:49`
- `.github/workflows/setup-tofu.yml:109`
- `.github/workflows/setup-tofu.yml:122`
- `.github/workflows/setup-tofu.yml:127`
- `.github/workflows/setup-tofu.yml:149`
- `.github/workflows/setup-tofu.yml:154`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to a mutable tag (@v4) rather than a full 40-character commit SHA. Failing references: actions/checkout@v4 appears in 7 steps in setup-tofu.yml and 1 step in retag.yml. A mutable tag can be silently redirected to a different (potentially malicious) commit.

Locations:

- `.github/workflows/setup-tofu.yml:26`
- `.github/workflows/setup-tofu.yml:43`
- `.github/workflows/setup-tofu.yml:62`
- `.github/workflows/setup-tofu.yml:88`
- `.github/workflows/setup-tofu.yml:117`
- `.github/workflows/setup-tofu.yml:143`
- `.github/workflows/setup-tofu.yml:165`
- `.github/workflows/retag.yml:12`

### permissions (severity: medium)

missing-permissions: setup-tofu.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs (tofu-version-files, tofu-versions, tofu-arguments, tofu-run-local, tofu-credentials-cloud, tofu-credentials-enterprise, tofu-credentials-none). The GITHUB_TOKEN is granted default permissions which may be broader than necessary.

Locations:

- `.github/workflows/setup-tofu.yml:1`

### permissions (severity: medium)

missing-permissions: retag.yml has no top-level permissions: key and no job-level permissions: key on the tag job, which performs git push --tag --force and requires write access.

Locations:

- `.github/workflows/retag.yml:1`

### permissions (severity: medium)

missing-permissions: continuous-integration.yml has no top-level permissions: key and no job-level permissions: key on either the check-dist or test jobs.

Locations:

- `.github/workflows/continuous-integration.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, permissions

**Notes:**

Fixed all findings across 3 workflow files:

1. setup-tofu.yml:
   - Added `permissions: {}` at top level
   - Pinned all 7 `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262 # v4`
   - Line 49: Moved `${{ matrix['tofu-versions'] }}` to env var `TOFU_VERSION`, used `grep "$TOFU_VERSION"` in shell
   - Line 109: Moved `${{ steps.plan.outputs.stdout }}` to env var `PLAN_STDOUT`, used `echo "$PLAN_STDOUT"`
   - Lines 122/127: Moved `${{ env.TF_CLOUD_API_TOKEN }}` to env var `TF_TOKEN` in each step, used `grep "token = \"$TF_TOKEN\""`
   - Lines 149/154: Same fix for tofu-credentials-enterprise job

2. retag.yml:
   - Added `permissions: contents: write` (required for git push --tag --force)
   - Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262 # v4`

3. continuous-integration.yml:
   - Added `permissions: contents: read` at top level

### Iteration 2

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded Terraform Cloud API token literal ('XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX') with '${{ secrets.TF_CLOUD_API_TOKEN }}' in two job-level env: blocks within .github/workflows/setup-tofu.yml: the tofu-credentials-cloud job (previously line 96) and the tofu-credentials-enterprise job (previously line 121). Both jobs now reference the token from GitHub Actions secrets instead of embedding a literal value.

