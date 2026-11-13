<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.4** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes a value from the github context to $GITHUB_ENV with no newline stripping: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. It should pass through an env var and be cleaned with `printf '%s' "$VAR" | tr -d '\n\r'` before the write. The PYTHON_VERSION line is computed inside the script and is fine.

Locations:

- `action.yml:663`

### script-injection (severity: high)

Rule (a): the 'Setup Environment Variables' step puts an expression straight into the run script: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`.

Locations:

- `action.yml:663`

### script-injection (severity: high)

Rule (a): the 'Install Dependencies' step puts expressions straight into the run script: `pip install -r "${{ github.action_path }}/requirements.txt"`, `if [ "${{ inputs.access-token }}" != "" ]`, `repo_url="git+https://${{ inputs.access-token }}@github.com/..."` and `pip install "${repo_url}/@${{ inputs.nuitka-version }}#egg=nuitka"`. Whoever sets inputs.nuitka-version or inputs.access-token controls the shell code that runs. These values should go through env: and be used as quoted variables.

Locations:

- `action.yml:668`
- `action.yml:671`
- `action.yml:672`
- `action.yml:677`

### script-injection (severity: high)

Rule (a): the 'Build with Nuitka' step puts the whole inputs object into a single-quoted shell string: `NUITKA_WORKFLOW_INPUTS=$(echo '${{ toJson(inputs) }}' | python -c ...)`. Any input that contains a single quote ends the string and can run any shell command. Pass `${{ toJson(inputs) }}` through an env: variable and use it as "$VAR".

Locations:

- `action.yml:703`

### unpinned-uses (severity: high)

A composite action step uses a mutable tag instead of a full commit SHA: `uses: actions/cache@v4`.

Locations:

- `action.yml:686`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml.j2 without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml.j2 b/action.yml.j2
index 08eb79a..6af2bff 100644
--- a/action.yml.j2
+++ b/action.yml.j2
@@ -122,22 +122,29 @@ runs:
     - name: Setup Environment Variables
       if: ${{ !inputs.disable-cache }}
       shell: bash
+      env:
+        ACTION_PATH: ${{ github.action_path }}
       run: |
-        echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV
+        safe_action_path=$(printf '%s' "$ACTION_PATH" | tr -d '\n\r')
+        echo "NUITKA_CACHE_DIR=${safe_action_path}/nuitka/cache" >> $GITHUB_ENV
         echo "PYTHON_VERSION=$(python --version | awk '{print $2}' | cut -d '.' -f 1,2)" >> $GITHUB_ENV
     - name: Install Dependencies
       shell: bash
+      env:
+        ACTION_PATH: ${{ github.action_path }}
+        ACCESS_TOKEN: ${{ inputs.access-token }}
+        NUITKA_VERSION: ${{ inputs.nuitka-version }}
       run: |
-        pip install -r "${{ github.action_path }}/requirements.txt"
+        pip install -r "$ACTION_PATH/requirements.txt"
 
         # With commercial access token, use that repository.
-        if [ "${{ inputs.access-token }}" != "" ]; then
-          repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"
+        if [ "$ACCESS_TOKEN" != "" ]; then
+          repo_url="git+https://$ACCESS_TOKEN@github.com/Nuitka/Nuitka-commercial.git"
         else
           repo_url="git+https://$@github.com/Nuitka/Nuitka.git"
         fi
 
-        pip install "${repo_url}/@${{ inputs.nuitka-version }}#egg=nuitka"
+        pip install "${repo_url}/@$NUITKA_VERSION#egg=nuitka"
     - name: Install ccache
       # TODO: Proper "in" test could make sense here.
       if: ${{ inputs.disable-cache != 'ccache' && runner.os == 'Linux' }}
@@ -146,7 +153,7 @@ runs:
         sudo apt-get install -y ccache
     - name: Cache Nuitka cache directory
       if: ${{ !inputs.disable-cache }}
-      uses: actions/cache@v4
+      uses: actions/cache@0057852bfaa89a56745cba8c7296529d2fc39830 # v4
       with:
         path: ${{ env.NUITKA_CACHE_DIR }}
         key: nuitka-${{ inputs.caching-key }}-${{ runner.os }}-${{ runner.arch }}-python-${{ env.PYTHON_VERSION }}-nuitka-${{ github.sha }}
@@ -159,11 +166,12 @@ runs:
       shell: bash
       env:
         PYTHONUTF8: 1
+        NUITKA_WORKFLOW_INPUTS_JSON: ${{ toJson(inputs) }}
       run: |
         set -e
 
         # Prepare the JSON string for Nuitka, filtering out action-specific keys using Python
-        NUITKA_WORKFLOW_INPUTS=$(echo '${{ toJson(inputs) }}' | python -c "import sys, json; data = json.load(sys.stdin); [data.pop(k, None) for k in ['nuitka-version', 'working-directory', 'access-token', 'disable-cache', 'caching-key']]; json.dump(data, sys.stdout, ensure_ascii=False)")
+        NUITKA_WORKFLOW_INPUTS=$(printf '%s' "$NUITKA_WORKFLOW_INPUTS_JSON" | python -c "import sys, json; data = json.load(sys.stdin); [data.pop(k, None) for k in ['nuitka-version', 'working-directory', 'access-token', 'disable-cache', 'caching-key']]; json.dump(data, sys.stdout, ensure_ascii=False)")
 
         # Pass the filtered JSON to Nuitka via an environment variable
         export NUITKA_WORKFLOW_INPUTS

```

Locations:

- `action.yml.j2`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 824b3fd..1c0096a 100644
--- a/action.yml
+++ b/action.yml
@@ -1,33 +1,25 @@
 # yamllint disable rule:line-length
 # yamllint disable rule:comments
 # too many spelling things, spell-checker: disable
----
 name: Build executables or extension modules from Python projects
-
 branding:
   icon: "package"
   color: "blue"
-
 description: "Build a modules or standalone executable from python code using Nuitka on Mac/Linux/Windows. Very wide range of compatibility."
-
 inputs:
   working-directory:
     default: "."
     description: "Directory to run nuitka in if not top level"
-
   nuitka-version:
     default: "main"
     description: "Version of nuitka to use, branches, tags work"
-
   ### Only Required Input ###
   script-name:
     required: true
     description: "Path to python script that is to be built."
-
   # Enable Nuitka Commercial features of Nuitka using a PAT (personal access token)
   access-token:
     description: "Github personal access token of an account authorized to access the Nuitka-commercial repo"
-
   ### Nuitka Modes ###
   mode:
     description: |
@@ -57,7 +49,6 @@ inputs:
     description: |
       Use debug version or not. Default uses what you are using to run Nuitka, most
       likely a non-debug version. Only for debugging and testing purposes.
-
   ### Nuitka Plugins to enable. ###
   enable-plugins:
     description: |
@@ -84,7 +75,6 @@ inputs:
       to provide extra decisions. Format is currently
       --module-parameter=module.name-option-name=value
       Default empty.
-
   ### Nuitka Tracing/Reporting features
   report:
     description: |
@@ -119,7 +109,6 @@ inputs:
     description: |
       Provide memory information and statistics.
       Defaults to off.
-
   ### Control the inclusion of data files in result. ###
   include-package-data:
     description: |
@@ -168,7 +157,6 @@ inputs:
       Include raw directories completely in the distribution. This is
       recursive. Check '--include-data-dir' to use the sane option.
       Default empty.
-
   ### Control the inclusion of modules and packages in result. ###
   include-package:
     description: 'Include a whole package. Give as a Python namespace, e.g. "some_package.sub_package" and Nuitka will then find it and include it and all the modules found below that disk location in the binary or extension module it creates, and make it available for import by the code. To avoid unwanted sub packages, e.g. tests you can e.g. do this "--nofollow-import-to=*.tests". Default empty.'
@@ -184,7 +172,6 @@ inputs:
     description: "Do not follow to that module name even if used, or if a package name, to the whole package in any case, overrides all other options. Can be given multiple times. Default empty."
   user-package-configuration-file:
     description: "User provided YAML file with package configuration. You can include DLLs, remove bloat, add hidden dependencies. Check User Manual for a complete description of the format to use. Can be given multiple times. Defaults to empty."
-
   ### Onefile behavior details ###
   onefile-tempdir-spec:
     description: "Use this as a folder to unpack onefile. Defaults to '%TEMP%\\onefile_%PID%_%TIME%', but e.g. '%CACHE_DIR%/%COMPANY%/%PRODUCT%/%VERSION%' would be cached and good too."
@@ -192,7 +179,6 @@ inputs:
     description: "When stopping the child, e.g. due to CTRL-C or shutdown, how much time to allow before killing it the hard way. Unit is ms. Default 5000."
   onefile-no-compression:
     description: "When creating the onefile, disable compression of the payload. Default is false."
-
   warn-implicit-exceptions:
     description: |
       Enable warnings for implicit exceptions detected at compile time.
@@ -211,7 +197,6 @@ inputs:
       certain topics, and typically point to the Nuitka website. The mnemonic is the part
       of the URL at the end, without the HTML suffix. Can be given multiple times and
       accepts shell pattern. Default empty.
-
   ### Deployment modes ###
   deployment:
     description: |
@@ -225,20 +210,17 @@ 
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, unpinned-uses, static-inline-injection, prior-fix

**Notes:**

Changes are in both action.yml and action.yml.j2. (1) Setup Environment Variables: github.action_path now comes in through the ACTION_PATH env var, and newlines are stripped with tr -d '\n\r' before the value is written to $GITHUB_ENV. (2) Install Dependencies: action_path, access-token and nuitka-version now come in through the env vars ACTION_PATH, ACCESS_TOKEN and NUITKA_VERSION. The script uses them as quoted variables. (3) actions/cache@v4 is pinned to commit 0057852bfaa89a56745cba8c7296529d2fc39830 (# v4). (4) Build with Nuitka: toJson(inputs) is passed through the NUITKA_WORKFLOW_INPUTS_JSON env var and sent to python with printf '%s' "$VAR". The previous harden's patch is re-applied in both files. The parts of that patch shown in the request were blank-line and separator removals in the inputs section, which don't affect security, so I didn't redo them.

