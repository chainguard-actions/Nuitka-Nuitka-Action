<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml, violating rule (a). In the 'Install Dependencies' step, the attacker-controlled inputs `${{ inputs.access-token }}` and `${{inputs.nuitka-version }}` are interpolated directly into shell commands. An attacker can inject shell metacharacters via these inputs. For example: `if [ "${{ inputs.access-token }}" != "" ]` and `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"`. In the 'Setup Environment Variables' step, `${{ github.action_path }}` is also interpolated directly in a run: block.

Locations:

- `action.yml:30459`
- `action.yml:30849`
- `action.yml:30923`
- `action.yml:31114`

### github-env-injection (severity: high)

The 'Setup Environment Variables' run: block writes a value derived from `${{ github.action_path }}` directly to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The line `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` writes an unsanitized expression to the special environment file.

Locations:

- `action.yml:30459`

### unpinned-uses (severity: high)

The action uses `actions/cache@v4` which is pinned to a mutable version tag rather than an immutable full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved. It should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

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
1. script-injection / static-inline-injection: Moved ${{ inputs.access-token }}, ${{ inputs.nuitka-version }}, and ${{ github.action_path }} out of run: shell strings and into env: blocks. Shell scripts now reference plain environment variables ($ACCESS_TOKEN, $NUITKA_VERSION, $ACTION_PATH).
2. github-env-injection: In the 'Setup Environment Variables' step, sanitized the ACTION_PATH value using `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV.
3. unpinned-uses: Pinned actions/cache@v4 to the full immutable commit SHA: actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4.

