<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.3** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes a github-context value to $GITHUB_ENV without the printf '%s' | tr -d '\n\r' cleanup step first: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. The value goes straight into the env file. It should be passed in through an env: variable, have newlines stripped, and be written to a quoted "$GITHUB_ENV".

Locations:

- `action.yml:639`

### script-injection (severity: high)

Rule (a): the 'Setup Environment Variables' step puts an expression directly inside the run: script: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. Use the $GITHUB_ACTION_PATH env var (quoted) instead.

Locations:

- `action.yml:639`

### script-injection (severity: high)

Rule (a): the 'Install Dependencies' step puts several expressions directly inside the run: script: `pip install -r "${{ github.action_path }}/requirements.txt"`, `if [ "${{ inputs.access-token }}" != "" ]; then`, `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` and `pip install "${repo_url}/@${{inputs.nuitka-version }}#egg=nuitka"`. The caller controls inputs.access-token and inputs.nuitka-version, so a crafted value can inject shell commands. Pass them in through env: variables and expand them in double quotes.

Locations:

- `action.yml:646`
- `action.yml:649`
- `action.yml:650`
- `action.yml:655`

### unpinned-uses (severity: high)

A composite step references an action by a mutable tag instead of a full 40-character commit SHA: `uses: actions/cache@v4`.

Locations:

- `action.yml:664`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index a45348d..306d8e0 100644
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
@@ -55,7 +47,6 @@ inputs:
     description: |
       Use debug version or not. Default uses what you are using to run Nuitka, most
       likely a non-debug version. Only for debugging and testing purposes.
-
   ### Nuitka Plugins to enable. ###
   enable-plugins:
     description: |
@@ -77,7 +68,6 @@ inputs:
       to provide extra decisions. Format is currently
       --module-parameter=module.name-option-name=value
       Default empty.
-
   ### Nuitka Tracing/Reporting features
   report:
     description: |
@@ -112,7 +102,6 @@ inputs:
     description: |
       Provide memory information and statistics.
       Defaults to off.
-
   ### Control the inclusion of data files in result. ###
   include-package-data:
     description: |
@@ -161,7 +150,6 @@ inputs:
       Include raw directories completely in the distribution. This is
       recursive. Check '--include-data-dir' to use the sane option.
       Default empty.
-
   ### Control the inclusion of modules and packages in result. ###
   include-package:
     description: 'Include a whole package. Give as a Python namespace, e.g. "some_package.sub_package" and Nuitka will then find it and include it and all the modules found below that disk location in the binary or extension module it creates, and make it available for import by the code. To avoid unwanted sub packages, e.g. tests you can e.g. do this "--nofollow-import-to=*.tests". Default empty.'
@@ -177,7 +165,6 @@ inputs:
     description: "Do not follow to that module name even if used, or if a package name, to the whole package in any case, overrides all other options. Can be given multiple times. Default empty."
   user-package-configuration-file:
     description: "User provided YAML file with package configuration. You can include DLLs, remove bloat, add hidden dependencies. Check User Manual for a complete description of the format to use. Can be given multiple times. Defaults to empty."
-
   ### Onefile behavior details ###
   onefile-tempdir-spec:
     description: "Use this as a folder to unpack onefile. Defaults to '%TEMP%\\onefile_%PID%_%TIME%', but e.g. '%CACHE_DIR%/%COMPANY%/%PRODUCT%/%VERSION%' would be cached and good too."
@@ -185,7 +172,6 @@ inputs:
     description: "When stopping the child, e.g. due to CTRL-C or shutdown, how much time to allow before killing it the hard way. Unit is ms. Default 5000."
   onefile-no-compression:
     description: "When creating the onefile, disable compression of the payload. Default is false."
-
   warn-implicit-exceptions:
     description: |
       Enable warnings for implicit exceptions detected at compile time.
@@ -204,7 +190,6 @@ inputs:
       certain topics, and typically point to the Nuitka website. The mnemonic is the part
       of the URL at the end, without the HTML suffix. Can be given multiple times and
       accepts shell pattern. Default empty.
-
   ### Deployment modes ###
   deployment:
     description: |
@@ -218,20 +203,17 @@ 
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, static-inline-injection, unpinned-uses, prior-fix

**Notes:**

All changes are in action.yml. I haven't run the action.

- **Setup Environment Variables:** the step now builds the cache path from $GITHUB_ACTION_PATH instead of the github.action_path expression. Newlines are stripped with printf | tr -d '\n\r' before it writes NUITKA_CACHE_DIR to "$GITHUB_ENV", which is now quoted. The PYTHON_VERSION write also uses the quoted file name.
- **Install Dependencies:** inputs.access-token and inputs.nuitka-version now come in through the env: block as INPUT_ACCESS_TOKEN and INPUT_NUITKA_VERSION, and are only used inside double quotes. The requirements path also uses $GITHUB_ACTION_PATH. Upstream's `$@` in the open-source repo URL is unchanged; it expands to nothing in a composite step, as before.
- **Pinning:** actions/cache@v4 is now pinned to 0057852bfaa89a56745cba8c7296529d2fc39830, with `# v4` kept as a comment.
- **Prior fix:** I could only see the start of the previous patch because it was cut off. That part removes blank lines and a `---` line, which aren't security changes. Its security changes appear to be the same as the fixes above, and those are now in place again.

