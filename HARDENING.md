<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action--/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Nuitka--Nuitka-Action--/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation in run: blocks. In the 'Setup Environment Variables' step, ${{ github.action_path }} is interpolated directly inside a run: shell command. In the 'Install Dependencies' step, ${{ github.action_path }}, ${{ inputs.access-token }}, and ${{ inputs.nuitka-version }} are all interpolated directly inside run: shell commands. Any ${{ ... }} expression inside a run: block is a script-injection risk as the value is substituted into the shell command string before the shell parses it. Offending lines include: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`, `pip install -r "${{ github.action_path }}/requirements.txt"`, `if [ "${{ inputs.access-token }}" != "" ]; then`, `repo_url="git+https://${{ inputs.access-token }}@github.com/..."`, and `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"`.

Locations:

- `action.yml:30354`
- `action.yml:30362`

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes ${{ github.action_path }} (a github.* context value) directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The offending line is: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. An attacker who can influence github.action_path could inject newlines to set arbitrary environment variables.

Locations:

- `action.yml:30354`

### unpinned-uses (severity: high)

The composite action uses actions/cache@v4, which is a mutable tag reference rather than a pinned full-length SHA commit hash. This means the action could be silently updated to a malicious version without any change to this file. It should be pinned to a full 40-character commit SHA (e.g., actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4).

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

Fixed all findings in action.yml:
1. script-injection + github-env-injection (Setup Environment Variables step): Moved ${{ github.action_path }} to an env var ACTION_PATH, then sanitized it with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV.
2. script-injection + static-inline-injection (Install Dependencies step): Moved ${{ github.action_path }}, ${{ inputs.access-token }}, and ${{ inputs.nuitka-version }} to env vars ACTION_PATH, ACCESS_TOKEN, and NUITKA_VERSION respectively. All shell commands now reference plain env vars.
3. unpinned-uses: Pinned actions/cache@v4 to full commit SHA actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

