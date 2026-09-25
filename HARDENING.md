<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ }}` expressions are directly interpolated inside `run:` shell command strings in action.yml, violating rule (a). In the 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` interpolates `github.action_path` directly into the shell. In the 'Install Dependencies' step: `pip install -r "${{ github.action_path }}/requirements.txt"` interpolates `github.action_path`; `if [ "${{ inputs.access-token }}" != "" ]` and `repo_url="git+https://${{ inputs.access-token }}@github.com/..."` interpolate the user-controlled `inputs.access-token` directly into shell; and `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"` interpolates `inputs.nuitka-version` directly. Any of these expressions could contain shell metacharacters that execute arbitrary commands.

Locations:

- `action.yml:30459`
- `action.yml:30729`
- `action.yml:30853`
- `action.yml:30927`
- `action.yml:31117`

### github-env-injection (severity: high)

In the 'Setup Environment Variables' step, the expression `${{ github.action_path }}` is written directly to `$GITHUB_ENV` via `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although `github.action_path` is typically controlled by GitHub, it is still an expression interpolated before the shell sees it, and writing unsanitized expressions to GITHUB_ENV can allow newline injection to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:30486`

### unpinned-uses (severity: high)

The composite action step 'Cache Nuitka cache directory' uses `actions/cache@v4`, which is a mutable tag reference rather than a pinned full 40-character SHA commit hash. This means the action could be silently updated to a different (potentially malicious) version without any change to this file, creating a supply-chain risk.

Locations:

- `action.yml:31465`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:647`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:648`

### static-inline-injection (severity: high)

shell injection: expression "${{inputs.nuitka-version }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:653`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:
1. Setup Environment Variables step: Moved `github.action_path` to an `env:` block as `ACTION_PATH`. Added sanitization with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to GITHUB_ENV to prevent newline injection.
2. Install Dependencies step: Moved `github.action_path` → `ACTION_PATH`, `inputs.access-token` → `ACCESS_TOKEN`, and `inputs.nuitka-version` → `NUITKA_VERSION` into the step's `env:` block. All three are now referenced as plain shell variables (`$ACTION_PATH`, `$ACCESS_TOKEN`, `$NUITKA_VERSION`) in the `run:` script.
3. Cache Nuitka cache directory step: Pinned `actions/cache@v4` to the full commit SHA `actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4`.

