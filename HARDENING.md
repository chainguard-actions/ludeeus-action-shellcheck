<!-- markdownlint-disable -->

# Hardening Report: ludeeus--action-shellcheck/2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ludeeus--action-shellcheck/2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in action.yaml. This causes YAML template substitution to inject values into the shell script before the shell parses them, enabling script injection.

- 'Download shellcheck' step: `if [[ "${{ runner.os }}" == "macOS" ]];` and `curl -Lso "${{ github.action_path }}/sc.tar.xz"`, `tar -xf "${{ github.action_path }}/sc.tar.xz" -C "${{ github.action_path }}"`, `mv "${{ github.action_path }}/shellcheck-...`
- 'Display shellcheck version' step: `"${{ github.action_path }}/shellcheck" --version`
- 'Run the check' step: `"${{ github.action_path }}/shellcheck"` (used twice)
- 'Exit action' step: `exit ${{steps.check.outputs.statuscode}}`

All ${{ ... }} expressions must be moved to env: vars and referenced as $ENV_VAR in the shell script.

Locations:

- `action.yaml:63`
- `action.yaml:69`
- `action.yaml:72`
- `action.yaml:73`
- `action.yaml:74`
- `action.yaml:78`
- `action.yaml:170`
- `action.yaml:174`
- `action.yaml:186`

### script-injection (severity: high)

Sub-rule (b): Multiple env vars holding user-controlled input values are expanded unquoted inside run: shell commands, allowing shell metacharacter injection (`;`, `|`, `&`, `$(...)`, glob chars, whitespace splitting).

- 'Gather excluded paths' step: `for path in ${INPUT_IGNORE}` and `for path in ${INPUT_IGNORE_PATHS}` and `for name in ${INPUT_IGNORE_NAMES}` — all unquoted iterations over inputs.ignore, inputs.ignore_paths, inputs.ignore_names.
- 'Gather additional files' step: `for file in ${INPUT_ADDITIONAL_FILES}` — unquoted iteration over inputs.additional_files.
- 'Run the check' step: `find "${INPUT_SCANDIR}" ${INPUT_EXCLUDE_ARGS} ...` — INPUT_EXCLUDE_ARGS (from steps.exclude.outputs.excludes) is unquoted; `${INPUT_ADDITIONAL_FILE_ARGS}` (from steps.additional.outputs.files) is unquoted; `${INPUT_SHELLCHECK_OPTIONS}` (from steps.options.outputs.options) is unquoted in both shellcheck invocations.

All these expansions must be double-quoted: "${VAR}".

Locations:

- `action.yaml:103`
- `action.yaml:109`
- `action.yaml:118`
- `action.yaml:126`
- `action.yaml:138`
- `action.yaml:163`
- `action.yaml:171`
- `action.yaml:175`

### github-env-injection (severity: high)

Three run: steps write values derived from user-controlled inputs to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker can inject newlines into these inputs to smuggle additional key=value pairs into GITHUB_OUTPUT, potentially overwriting outputs consumed by downstream steps.

1. 'Set options' step: `echo "options=${options[@]}" >> $GITHUB_OUTPUT` — options array is built from inputs.severity (INPUT_SEVERITY) and inputs.format (INPUT_FORMAT) without sanitization.

2. 'Gather excluded paths' step: `echo "excludes=${excludes[@]}" >> $GITHUB_OUTPUT` — excludes array is built from inputs.ignore (INPUT_IGNORE), inputs.ignore_paths (INPUT_IGNORE_PATHS), and inputs.ignore_names (INPUT_IGNORE_NAMES) without sanitization.

3. 'Gather additional files' step: `echo "files=${files[@]}" >> $GITHUB_OUTPUT` — files array is built from inputs.additional_files (INPUT_ADDITIONAL_FILES) without sanitization.

Locations:

- `action.yaml:89`
- `action.yaml:118`
- `action.yaml:128`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings in action.yaml:

1. script-injection (a): Moved all ${{ runner.os }}, ${{ github.action_path }}, and ${{ steps.check.outputs.statuscode }} expressions from run: shell blocks into env: blocks (as RUNNER_OS, ACTION_PATH, STATUSCODE), and replaced them with proper $ENV_VAR references in the shell scripts.

2. script-injection (b): Replaced all unquoted list expansions (for path in ${INPUT_IGNORE}, etc.) with xargs-based tokenization using `while IFS= read -r -d '' token; do ... done < <(printf '%s' "$VAR" | xargs printf '%s\0')` with proper guards. In the 'Run the check' step, INPUT_EXCLUDE_ARGS, INPUT_ADDITIONAL_FILE_ARGS, and INPUT_SHELLCHECK_OPTIONS are now tokenized into bash arrays and expanded with "${array[@]}" quoting.

3. github-env-injection: All three GITHUB_OUTPUT writes (in 'Set options', 'Gather excluded paths', 'Gather additional files' steps) now sanitize values with `printf '%s' "${var[*]}" | tr -d '\n\r'` before writing to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

1. unpinned-uses: Replaced `actions/checkout@v3` with pinned SHA `actions/checkout@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3` in all 5 workflow files (.github/workflows/additional_files.yml, check_together.yml, ignore_names.yml, ignore_paths.yml, scandir.yml). 2. script-injection: Moved all `${{ steps.*.outputs.files }}` expressions from run: shell strings into env: blocks (CHECK_FILES, ONE_FILES, TWO_FILES) and updated shell scripts to reference plain environment variables. 3. github-env-injection: In action.yaml line 175, added `safe_filepaths=$(printf '%s' "${filepaths[*]}" | tr -d '\n\r')` before writing to GITHUB_OUTPUT to strip newline characters that could enable injection of arbitrary key=value pairs.

