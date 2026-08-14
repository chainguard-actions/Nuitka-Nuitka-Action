<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple GitHub Actions expressions are directly interpolated inside `run:` shell command strings, enabling script injection.

1. In the 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV` — `${{ github.action_path }}` is interpolated directly in the shell command.

2. In the 'Install Dependencies' step:
   - `pip install -r "${{ github.action_path }}/requirements.txt"` — expression in shell string
   - `if [ "${{ inputs.access-token }}" != "" ]; then` — attacker-controlled input directly in shell
   - `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` — attacker-controlled input injected into URL/shell variable assignment
   - `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"` — attacker-controlled input directly in shell

An attacker supplying a malicious `inputs.access-token` or `inputs.nuitka-version` value (e.g. containing shell metacharacters or newlines) can achieve arbitrary command execution. All `${{ ... }}` expressions must be moved to `env:` variables and those variables must be double-quoted in the shell script.

Locations:

- `action.yml:255`
- `action.yml:260`
- `action.yml:263`
- `action.yml:264`
- `action.yml:268`

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes a value derived from `${{ github.action_path }}` directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is:

  `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`

Although `github.action_path` is not directly attacker-controlled, any `${{ ... }}` expression written to `$GITHUB_ENV` without newline sanitization can allow environment variable injection if the value contains newlines. The fix is to capture the value into a shell variable, sanitize it with `printf '%s' "$VAR" | tr -d '\n\r'`, and then write the sanitized value.

Locations:

- `action.yml:255`

### unpinned-uses (severity: high)

The composite action step uses `actions/cache@v3`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. A supply-chain attacker who compromises the `actions/cache` repository could push a malicious commit under the `v3` tag and have it executed by all users of this action. Fix: pin to a full SHA, e.g. `actions/cache@6849a6489940f00c2f30c0fb92c6274307ccb58a # v4`.

Locations:

- `action.yml:277`

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
1. script-injection / static-inline-injection: Moved all ${{ }} expressions (${{ github.action_path }}, ${{ inputs.access-token }}, ${{ inputs.nuitka-version }}) out of run: blocks into env: blocks (ACTION_PATH, ACCESS_TOKEN, NUITKA_VERSION). Shell scripts now reference plain environment variables.
2. github-env-injection: The action_path value is sanitized with `printf '%s' "$ACTION_PATH" | tr -d '\n\r'` before being written to $GITHUB_ENV.
3. unpinned-uses: Pinned actions/cache@v3 to its full commit SHA actions/cache@6f8efc29b200d32929f49075959781ed54ec270c # v3.

