<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Install Dependencies' run: block directly interpolates attacker-controlled input expressions into shell commands without routing through env: variables. Specifically: (a) `${{ inputs.access-token }}` is interpolated directly into a shell `if` condition and into a URL string (`git+https://${{ inputs.access-token }}@github.com/...`), and (b) `${{ inputs.nuitka-version }}` is interpolated directly into a `pip install` command. An attacker who controls these inputs (e.g. via workflow_dispatch or by calling this composite action) can inject arbitrary shell commands. Additionally, `${{ github.action_path }}` is interpolated directly in run: blocks in both the 'Setup Environment Variables' and 'Install Dependencies' steps — any ${{ }} expression inside a run: shell string is a script-injection finding per rule (a).

Locations:

- `action.yml:30459`
- `action.yml:30729`
- `action.yml:30849`
- `action.yml:30923`
- `action.yml:31114`

### github-env-injection (severity: high)

The 'Setup Environment Variables' run: block writes `${{ github.action_path }}` directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The line `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` interpolates a ${{ }} expression and writes it to the special environment file without stripping newlines, which could allow environment variable injection.

Locations:

- `action.yml:30459`

### unpinned-uses (severity: high)

The composite action uses `actions/cache@v4` — a mutable tag reference rather than a pinned full 40-character SHA commit hash. This means the action could be silently updated to a malicious version without the consuming workflow's knowledge, enabling a supply-chain attack.

Locations:

- `action.yml:31471`

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
1. script-injection / static-inline-injection: Moved ${{ github.action_path }}, ${{ inputs.access-token }}, and ${{ inputs.nuitka-version }} out of run: shell strings into env: maps (ACTION_PATH, ACCESS_TOKEN, NUITKA_VERSION). Shell scripts now reference plain environment variables.
2. github-env-injection: The 'Setup Environment Variables' step now sanitizes ACTION_PATH with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV.
3. unpinned-uses: Pinned actions/cache@v4 to full SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

