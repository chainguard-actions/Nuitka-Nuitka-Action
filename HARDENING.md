<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Setup Environment Variables' run: block directly interpolates ${{ github.action_path }} into the shell command string. Although github.action_path is not attacker-controlled in the same way as inputs.*, any ${{ ... }} expression inside a run: block is a script-injection finding because the value flows through YAML template substitution before the shell ever sees it. Offending line: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`

Locations:

- `action.yml:637`

### script-injection (severity: high)

Sub-rule (a): The 'Install Dependencies' run: block directly interpolates multiple untrusted expressions into shell commands: (1) `${{ github.action_path }}` in `pip install -r "${{ github.action_path }}/requirements.txt"`, (2) `${{ inputs.access-token }}` in the if-condition and in the repo_url construction `git+https://${{ inputs.access-token }}@github.com/...`, and (3) `${{inputs.nuitka-version }}` in `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"`. The access-token and nuitka-version inputs are attacker-controllable and allow arbitrary command injection.

Locations:

- `action.yml:643`

### github-env-injection (severity: high)

The 'Setup Environment Variables' run: block writes a value derived from ${{ github.action_path }} directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Offending line: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. A newline injected into github.action_path could allow an attacker to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:637`

### unpinned-uses (severity: high)

The action uses `actions/cache@v4`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. It should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yml:659`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed all findings in action.yml:
1. Setup Environment Variables step: moved `${{ github.action_path }}` to env block as `ACTION_PATH`, then sanitized with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to GITHUB_ENV (fixes both script-injection and github-env-injection).
2. Install Dependencies step: moved `${{ github.action_path }}`, `${{ inputs.access-token }}`, and `${{ inputs.nuitka-version }}` to env block as `ACTION_PATH`, `ACCESS_TOKEN`, and `NUITKA_VERSION` respectively, then referenced them as plain shell variables (fixes script-injection and static-inline-injection findings).
3. Pinned `actions/cache@v4` to full commit SHA `actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4` (fixes unpinned-uses).

