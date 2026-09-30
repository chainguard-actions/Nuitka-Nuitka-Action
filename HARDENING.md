<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Setup Environment Variables' run: block directly interpolates ${{ github.action_path }} into a shell command string. The 'Install Dependencies' run: block directly interpolates attacker-controlled inputs ${{ inputs.access-token }} (used twice in shell conditionals and URL construction) and ${{inputs.nuitka-version}} (embedded in a pip install URL) into shell commands. Any ${{ }} expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value, allowing metacharacters to break out of the intended command context. These should be moved to env: variables and then double-quoted in the shell script.

Locations:

- `action.yml:541`
- `action.yml:548`

### github-env-injection (severity: high)

The 'Setup Environment Variables' run: block writes ${{ github.action_path }} directly to $GITHUB_ENV without sanitization: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. Although github.action_path is not directly attacker-controlled, the check requires that any expression written to GITHUB_ENV, GITHUB_OUTPUT, or GITHUB_PATH must be sanitized with `printf '%s' ... | tr -d '\n\r'` before the write to prevent newline injection attacks that could poison subsequent environment variable reads.

Locations:

- `action.yml:541`

### unpinned-uses (severity: high)

The composite action uses `actions/cache@v4` which is pinned to a mutable version tag rather than an immutable full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, making this a supply-chain risk. It should be pinned to a full SHA, e.g. `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

Locations:

- `action.yml:560`

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

Fixed all findings in action.yml:
1. 'Setup Environment Variables' step: Moved `${{ github.action_path }}` to env var `ACTION_PATH`, added newline sanitization with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to GITHUB_ENV, and quoted `$GITHUB_ENV`.
2. 'Install Dependencies' step: Moved `${{ github.action_path }}`, `${{ inputs.access-token }}`, and `${{inputs.nuitka-version }}` to env vars `ACTION_PATH`, `ACCESS_TOKEN`, and `NUITKA_VERSION` respectively, and referenced them as plain shell variables in the run block.
3. Pinned `actions/cache@v4` to full commit SHA `0057852bfaa89a56745cba8c7296529d2fc39830 # v4`.

