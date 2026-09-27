<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml, violating rule (a).

1. "Setup Environment Variables" step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` — `${{ github.action_path }}` is interpolated directly in the shell command.

2. "Install Dependencies" step:
   - `pip install -r "${{ github.action_path }}/requirements.txt"` — direct expression interpolation.
   - `if [ "${{ inputs.access-token }}" != "" ]; then` — attacker-controlled `inputs.access-token` interpolated directly in a shell conditional.
   - `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` — attacker-controlled `inputs.access-token` interpolated directly into a URL/variable assignment.
   - `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"` — attacker-controlled `inputs.nuitka-version` interpolated directly in a pip install command.

All of these allow shell metacharacter injection before the shell ever sees the value.

Locations:

- `action.yml:557`
- `action.yml:563`
- `action.yml:566`
- `action.yml:567`
- `action.yml:571`

### github-env-injection (severity: high)

The "Setup Environment Variables" run: block writes a value derived from the `github.*` context directly to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. A newline injected into `github.action_path` could allow an attacker to inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:557`

### unpinned-uses (severity: high)

The composite action step "Cache Nuitka cache directory" references `actions/cache@v4`, which uses a mutable tag (`v4`) rather than a pinned 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to point to malicious code. It should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yml:578`

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
1. 'Setup Environment Variables' step: moved github.action_path to env: block as ACTION_PATH, added sanitization with printf/tr before writing to GITHUB_ENV.
2. 'Install Dependencies' step: moved github.action_path, inputs.access-token, and inputs.nuitka-version to env: block as ACTION_PATH, ACCESS_TOKEN, and NUITKA_VERSION; shell script now references plain env vars.
3. Pinned actions/cache@v4 to full SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

