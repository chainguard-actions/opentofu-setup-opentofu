<!-- markdownlint-disable -->

# Hardening Report: opentofu--setup-opentofu/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **opentofu--setup-opentofu/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference `actions/checkout@v6` using a mutable tag instead of a full 40-character commit SHA. This allows the referenced action to be silently replaced with a different (potentially malicious) version. Affected references:
- `.github/workflows/retag.yml`: `uses: actions/checkout@v6`
- `.github/workflows/setup-tofu.yml`: `uses: actions/checkout@v6` (appears 6 times across all jobs)

All should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/retag.yml:14`
- `.github/workflows/setup-tofu.yml:26`
- `.github/workflows/setup-tofu.yml:38`
- `.github/workflows/setup-tofu.yml:65`
- `.github/workflows/setup-tofu.yml:80`
- `.github/workflows/setup-tofu.yml:97`
- `.github/workflows/setup-tofu.yml:113`

### script-injection (severity: high)

Two `run:` steps in `.github/workflows/setup-tofu.yml` directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a). This means the expression value is substituted into the shell command string before the shell parses it, allowing shell metacharacters to be injected.

1. `run: tofu version | grep ${{ matrix['tofu-versions']}}` — the matrix value is interpolated unquoted directly into the shell command. An attacker-controlled matrix value could inject shell metacharacters.

2. `run: echo "${{ steps.plan.outputs.stdout }}"` — the step output is interpolated directly into the shell command string. Step outputs can contain attacker-controlled content (e.g. from a `tofu plan` run against attacker-supplied configuration).

Fix: route values through `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/setup-tofu.yml:52`
- `.github/workflows/setup-tofu.yml:91`

### missing-permissions (severity: medium)

None of the workflow files declare a `permissions:` block at the top level or at the job level. Without explicit permissions, workflows run with the default repository permissions (which may include `write` access to contents, packages, etc.), violating the principle of least privilege.

- `.github/workflows/retag.yml`: no `permissions:` at top-level or job level; the `tag` job pushes tags and needs only `contents: write`.
- `.github/workflows/setup-tofu.yml`: no `permissions:` at top-level or any of its six jobs.
- `.github/workflows/continuous-integration.yml`: no `permissions:` at top-level or job level.

Locations:

- `.github/workflows/retag.yml:1`
- `.github/workflows/setup-tofu.yml:1`
- `.github/workflows/continuous-integration.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all 7 occurrences of `actions/checkout@v6` to the full commit SHA `df4cb1c069e1874edd31b4311f1884172cec0e10` with `# v6` comment for readability. Affected files: retag.yml (1 occurrence), setup-tofu.yml (6 occurrences).

2. **script-injection**: Fixed both injection points in setup-tofu.yml by moving `${{ ... }}` expressions into `env:` blocks and referencing them as plain shell variables:
   - `grep ${{ matrix['tofu-versions']}}` → `env: TOFU_VERSION: ${{ matrix['tofu-versions'] }}` + `grep "$TOFU_VERSION"`
   - `echo "${{ steps.plan.outputs.stdout }}"` → `env: PLAN_STDOUT: ${{ steps.plan.outputs.stdout }}` + `echo "$PLAN_STDOUT"`

3. **missing-permissions**: Added top-level `permissions:` blocks to all three workflow files:
   - retag.yml: `contents: write` (required to push tags)
   - setup-tofu.yml: `contents: read` (read-only checkout)
   - continuous-integration.yml: `contents: read` (read-only CI checks)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script injection findings in hardened/action/.github/workflows/setup-tofu.yml. In each affected step ('Validate OpenTofu Credentials (Windows)' and 'Validate Teraform Credentials (Linux & macOS)' in both tofu-credentials-cloud and tofu-credentials-enterprise jobs), the `${{ env.TF_CLOUD_API_TOKEN }}` expression was moved from the inline `run:` shell string into the step's `env:` block as `TOKEN: ${{ env.TF_CLOUD_API_TOKEN }}`. The shell scripts now reference the value via the plain environment variable `$TOKEN`, preventing YAML template substitution from enabling script injection.

### Iteration 1

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded literal token value 'XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' in TF_CLOUD_API_TOKEN with ${{ secrets.TF_CLOUD_API_TOKEN }} in both the tofu-credentials-cloud job (line 119) and the tofu-credentials-enterprise job (line 155) of .github/workflows/setup-tofu.yml. The validation steps that grep for the token value in config files will now use the secret value at runtime.

