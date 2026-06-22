<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **Nuitka--Nuitka-Action/v1.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yml (sub-rule a). This allows template substitution before the shell sees the value, enabling script injection.

1. 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` — ${{ github.action_path }} is interpolated directly in the run: block.

2. 'Install Dependencies' step: `pip install -r "${{ github.action_path }}/requirements.txt"` — ${{ github.action_path }} interpolated directly.

3. 'Install Dependencies' step: `if [ "${{ inputs.access-token }}" != "" ]; then` — attacker-controlled input interpolated directly.

4. 'Install Dependencies' step: `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` — attacker-controlled input interpolated directly, allowing credential injection and command injection.

5. 'Install Dependencies' step: `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"` — attacker-controlled input interpolated directly into a pip install command.

Locations:

- `action.yml:255`
- `action.yml:260`
- `action.yml:263`
- `action.yml:264`
- `action.yml:269`

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes a value derived from ${{ github.action_path }} directly to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). The offending line is: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. While github.action_path is not directly attacker-controlled, the ${{ }} expression is interpolated before the shell runs, and the value is written to GITHUB_ENV without newline sanitization, which could allow environment variable injection if the path contains newlines.

Locations:

- `action.yml:255`

### unpinned-uses (severity: high)

The composite action uses `actions/cache@v3` which is pinned to a mutable tag (`v3`) rather than an immutable full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved to a different (potentially malicious) commit. It should be pinned to a full SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4`.

Locations:

- `action.yml:275`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:219`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.access-token }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:220`

### static-inline-injection (severity: high)

shell injection: expression "${{inputs.nuitka-version }}" appears directly in run: block of step "Install Dependencies"; move to env: map

Locations:

- `action.yml:225`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. script-injection & static-inline-injection: Moved all ${{ }} expressions (${{ github.action_path }}, ${{ inputs.access-token }}, ${{ inputs.nuitka-version }}) from run: blocks into env: blocks in both 'Setup Environment Variables' and 'Install Dependencies' steps. Shell scripts now reference plain environment variables ($ACTION_PATH, $ACCESS_TOKEN, $NUITKA_VERSION).
2. github-env-injection: Added newline sanitization using `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before writing to $GITHUB_ENV in the 'Setup Environment Variables' step.
3. unpinned-uses: Pinned actions/cache@v3 to its full commit SHA actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3.

