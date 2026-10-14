<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.5** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell commands in action.yml, violating rule (a). This allows expression values to be parsed as shell code before the shell ever sees them.

1. 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` — ${{ github.action_path }} interpolated directly in shell.

2. 'Install Dependencies' step: `pip install -r "${{ github.action_path }}/requirements.txt"`, `if [ "${{ inputs.access-token }}" != "" ]`, `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"`, and `pip install "${repo_url}/@${{ inputs.nuitka-version }}#egg=nuitka"` — inputs.access-token and inputs.nuitka-version are attacker-controllable and interpolated directly into shell commands.

3. 'Build with Nuitka' step: `NUITKA_WORKFLOW_INPUTS=$(echo '${{ toJson(inputs) }}' | python -c ...)` — toJson(inputs) (all workflow inputs) is interpolated directly into a shell command string.

Locations:

- `action.yml:1060`
- `action.yml:1066`
- `action.yml:1068`
- `action.yml:1069`
- `action.yml:1072`
- `action.yml:1090`

### github-env-injection (severity: high)

In the 'Setup Environment Variables' step, the value of ${{ github.action_path }} is written directly to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. Although github.action_path is not directly attacker-controlled, it is a github.* context value and the check requires sanitization before any write to GITHUB_ENV from such a source.

Locations:

- `action.yml:1060`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:855`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:856`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.nuitka-version }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:861`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all findings in hardened/action/action.yml:

1. Setup Environment Variables step: Moved ${{ github.action_path }} to env: block as ACTION_PATH. Added sanitization with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV, fixing both script-injection and github-env-injection findings.

2. Install Dependencies step: Moved ${{ github.action_path }}, ${{ inputs.access-token }}, and ${{ inputs.nuitka-version }} to env: block as ACTION_PATH, ACCESS_TOKEN, and NUITKA_VERSION respectively. All shell references now use plain env vars.

3. Build with Nuitka step: Moved ${{ toJson(inputs) }} to env: block as NUITKA_WORKFLOW_INPUTS_JSON. Changed echo pipe to `printf '%s' "$NUITKA_WORKFLOW_INPUTS_JSON"` to safely pass JSON to Python without shell interpretation.

