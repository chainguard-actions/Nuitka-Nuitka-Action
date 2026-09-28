<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings, violating rule (a). In the 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` embeds github.action_path directly in the shell. In the 'Install Dependencies' step: (1) `pip install -r "${{ github.action_path }}/requirements.txt"` embeds github.action_path; (2) `if [ "${{ inputs.access-token }}" != "" ]` embeds the user-controlled access-token input; (3) `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` embeds access-token in a URL — an attacker-controlled token value could inject shell metacharacters; (4) `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"` embeds the user-controlled nuitka-version input. All of these allow an attacker to inject arbitrary shell commands by supplying crafted input values.

Locations:

- `action.yml:563`
- `action.yml:568`
- `action.yml:572`
- `action.yml:573`
- `action.yml:577`

### github-env-injection (severity: high)

In the 'Setup Environment Variables' run: block, the value `${{ github.action_path }}` is written directly to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The line `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` allows a newline character in the path to inject additional environment variable assignments, potentially overwriting security-sensitive variables for subsequent steps.

Locations:

- `action.yml:563`

### unpinned-uses (severity: high)

The composite action uses `actions/cache@v4`, which is a mutable tag reference rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `actions/cache@5a3ec84eff668545956fd18022155c47e93e2684 # v4`.

Locations:

- `action.yml:580`

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
1. script-injection & github-env-injection: In 'Setup Environment Variables' step, moved github.action_path to env: block as ACTION_PATH, then sanitized with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV.
2. script-injection & static-inline-injection: In 'Install Dependencies' step, moved github.action_path → ACTION_PATH, inputs.access-token → ACCESS_TOKEN, and inputs.nuitka-version → NUITKA_VERSION into the step's env: block. Shell script now references plain environment variables.
3. unpinned-uses: Pinned actions/cache@v4 to full commit SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

