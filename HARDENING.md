<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.4** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml, allowing script injection.

1. 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` — ${{ github.action_path }} is expanded by the template engine before the shell sees it.

2. 'Install Dependencies' step:
   - `pip install -r "${{ github.action_path }}/requirements.txt"` — direct interpolation of github.action_path
   - `if [ "${{ inputs.access-token }}" != "" ]; then` — direct interpolation of attacker-controllable inputs.access-token
   - `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` — direct interpolation of inputs.access-token into a URL
   - `pip install "${repo_url}/@${{ inputs.nuitka-version }}#egg=nuitka"` — direct interpolation of inputs.nuitka-version

3. 'Build with Nuitka' step: `NUITKA_WORKFLOW_INPUTS=$(echo '${{ toJson(inputs) }}' | python -c ...)` — ${{ toJson(inputs) }} expands all inputs (including attacker-controlled values) directly into the shell command string before the shell parses it.

Locations:

- `action.yml:519`
- `action.yml:525`
- `action.yml:528`
- `action.yml:529`
- `action.yml:533`
- `action.yml:551`

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes ${{ github.action_path }} directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The line `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` interpolates the github.action_path context value directly into the environment file. Although github.action_path is typically controlled by GitHub, it is still a ${{ }} context value that flows through YAML template substitution and must be sanitized before being written to $GITHUB_ENV.

Locations:

- `action.yml:519`

### unpinned-uses (severity: high)

The composite action uses `actions/cache@v4` which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. It should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yml:537`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:671`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:672`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.nuitka-version }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:677`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:
1. Setup Environment Variables step: moved github.action_path to env: ACTION_PATH, added sanitization with printf/tr before writing to $GITHUB_ENV.
2. Install Dependencies step: moved github.action_path, inputs.access-token, and inputs.nuitka-version to env: block (ACTION_PATH, ACCESS_TOKEN, NUITKA_VERSION); shell script now uses plain $VAR references.
3. Build with Nuitka step: moved toJson(inputs) to env: NUITKA_WORKFLOW_INPUTS_JSON; shell script reads from that env var using printf instead of direct template interpolation.
4. Cache Nuitka cache directory step: pinned actions/cache@v4 to actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/action.yml.j2:

1. github-env-injection: Moved `${{ github.action_path }}` into an env: block as ACTION_PATH, then sanitized with `safe_action_path=$(printf '%s' "$ACTION_PATH" | tr -d '\n\r')` before writing to $GITHUB_ENV.

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks - github.action_path → ACTION_PATH, inputs.access-token → ACCESS_TOKEN, inputs.nuitka-version → NUITKA_VERSION, toJson(inputs) → NUITKA_WORKFLOW_INPUTS_JSON. The shell scripts now reference only plain environment variables.

3. unpinned-uses: Pinned actions/cache@v4 to full commit SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 with # v4 comment for readability.

