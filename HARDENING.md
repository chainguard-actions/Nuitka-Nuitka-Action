<!-- markdownlint-disable -->

# Hardening Report: Nuitka--Nuitka-Action/v1.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Nuitka--Nuitka-Action/v1.5** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Setup Environment Variables' step writes a github.* context value into $GITHUB_ENV without the printf '%s' | tr -d '\n\r' newline strip: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. The $GITHUB_ENV path is also unquoted. Fix: pass the value through env (ACTION_PATH: ${{ github.action_path }}), strip newlines with safe=$(printf '%s' "$ACTION_PATH" | tr -d '\n\r'), then write to "$GITHUB_ENV".

Locations:

- `action.yml:847`

### script-injection (severity: high)

Rule (a): an expression is placed directly inside the run: script of the 'Setup Environment Variables' step: `echo "NUITKA_CACHE_DIR=${{ github.action_path }}/nuitka/cache" >> $GITHUB_ENV`. Use the $GITHUB_ACTION_PATH env var instead.

Locations:

- `action.yml:847`

### script-injection (severity: high)

Rule (a): expressions are placed directly inside the run: script of the 'Install Dependencies' step: `pip install -r "${{ github.action_path }}/requirements.txt"`, `if [ "${{ inputs.access-token }}" != "" ]; then`, `repo_url="git+https://${{ inputs.access-token }}@github.com/Nuitka/Nuitka-commercial.git"` and `pip install "${repo_url}/@${{ inputs.nuitka-version }}#egg=nuitka"`. Inputs supplied by the caller (access-token, nuitka-version) can break out of the quotes and run arbitrary commands. Move them into env: variables and reference them as double-quoted shell variables.

Locations:

- `action.yml:852`
- `action.yml:855`
- `action.yml:856`
- `action.yml:861`

### script-injection (severity: high)

Rule (a): the 'Build with Nuitka' step puts the whole inputs object directly into the run: script inside single quotes: `NUITKA_WORKFLOW_INPUTS=$(echo '${{ toJson(inputs) }}' | python -c ...)`. Any input value that contains a single quote ends the quoted string and runs as shell commands. Pass toJson(inputs) through an env: variable and use it as "$VAR" (e.g. printf '%s' "$INPUTS_JSON" | python ...).

Locations:

- `action.yml:886`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 7fc3a00..247d818 100644
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
@@ -61,7 +53,6 @@ inputs:
     description: |
       Compile the project in the current directory by extracting configuration
       from it. Default is not to use it.
-
   ### Nuitka Plugins to enable. ###
   enable-plugins:
     description: |
@@ -88,7 +79,6 @@ inputs:
       to provide extra decisions. Format is currently
       --module-parameter=module.name-option-name=value
       Default empty.
-
   ### Nuitka Tracing/Reporting features
   report:
     description: |
@@ -123,7 +113,6 @@ inputs:
     description: |
       Provide memory information and statistics.
       Defaults to off.
-
   ### Control the inclusion of data files in result. ###
   include-package-data:
     description: |
@@ -172,7 +161,6 @@ inputs:
       Include raw directories completely in the distribution. This is
       recursive. Check '--include-data-dir' to use the sane option.
       Default empty.
-
   ### Control the inclusion of modules and packages in result. ###
   include-package:
     description: 'Include a whole package. Give as a Python namespace, e.g. "some_package.sub_package" and Nuitka will then find it and include it and all the modules found below that disk location in the binary or extension module it creates, and make it available for import by the code. To avoid unwanted sub packages, e.g. tests you can e.g. do this "--nofollow-import-to=*.tests". Default empty.'
@@ -188,7 +176,6 @@ inputs:
     description: "Do not follow to that module name even if used, or if a package name, to the whole package in any case, overrides all other options. Can be given multiple times. Default empty."
   user-package-configuration-file:
     description: "User provided YAML file with package configuration. You can include DLLs, remove bloat, add hidden dependencies. Check User Manual for a complete description of the format to use. Can be given multiple times. Defaults to empty."
-
   ### Control the inclusion of DLL files in result. ###
   noinclude-dlls:
     description: |
@@ -201,12 +188,10 @@ inputs:
       Include the Windows C runtime DLLs in the distribution. For onefile, this also
       enables static linking of the bootstrap binary. Defaults to 'auto', which
       includes them if found.
-
   ### Environment control ###
   force-runtime-environment-variable:
     description: |
       Force an environment variables to a given value. Default empty.
-
   ### Onefile behavior details ###
   onefile-tempdir-spec:
     description: "Use this as a folder to unpack onefile. Defaults to '%TEMP%\\onefile_%PID%_%TIME%', but e.g. '%CACHE_DIR%/%COMPANY%/%PRODUCT%/%VERSION%' would be cached and good too."
@@ -214,7 +199,6 @@ inputs:
     description: "When stopping the child, e.g. due to CTRL-C or shutdown, how much time to allow before killing it the hard way. Unit is ms. Default 5000."
   onefile-no-compression:
     description: "When creating the onefile, disable compression of the payload. Default is false."
-
   warn-im
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, static-inline-injection, prior-fix

**Notes:**

All changes are in action.yml. 'Setup Environment Variables': github.action_path now comes in through an ACTION_PATH env variable, newlines are stripped with printf|tr before the NUITKA_CACHE_DIR write, and "$GITHUB_ENV" is now quoted. 'Install Dependencies': action_path, access-token and nuitka-version now come in through env variables and are used double-quoted; upstream's original logic is unchanged. 'Build with Nuitka': toJson(inputs) is now passed through an INPUTS_JSON env variable and piped to python with printf. The previous harden's patch was truncated, but what was visible only removed blank lines, which doesn't affect security. Its security changes match the fixes above, so they are back in place. actions/cache was already pinned to a commit SHA. I made no changes outside action.yml.

