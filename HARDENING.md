# Hardening Report: ludeeus--action-shellcheck/2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **ludeeus--action-shellcheck/2.0.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Download shellcheck' run: block directly interpolates ${{ github.action_path }} inside shell commands (e.g., `curl -Lso "${{ github.action_path }}/sc.tar.xz"`, `tar -xf "${{ github.action_path }}/sc.tar.xz"`) without first assigning to an env var. All github.* expressions are considered attacker-controlled per the check rules.

Locations:

- `action.yaml:72`
- `action.yaml:75`

### script-injection (severity: high)

The 'Display shellcheck version' run: block directly interpolates ${{ github.action_path }} inside a shell command: `"${{ github.action_path }}/shellcheck" --version` without first assigning to an env var.

Locations:

- `action.yaml:82`

### script-injection (severity: high)

The 'Run the check' run: block directly interpolates ${{ github.action_path }} inside shell commands (e.g., `"${{ github.action_path }}/shellcheck"`) without first assigning to an env var.

Locations:

- `action.yaml:163`
- `action.yaml:167`

### github-env-injection (severity: high)

The 'Set options' step writes attacker-controlled inputs (inputs.severity via INPUT_SEVERITY, inputs.format via INPUT_FORMAT) to $GITHUB_OUTPUT without the required sanitization (printf '%s' ... | tr -d '\n\r'). An attacker can inject newlines to poison GITHUB_OUTPUT.

Locations:

- `action.yaml:96`

### github-env-injection (severity: high)

The 'Gather excluded paths' step writes attacker-controlled inputs (inputs.ignore via INPUT_IGNORE, inputs.ignore_paths via INPUT_IGNORE_PATHS, inputs.ignore_names via INPUT_IGNORE_NAMES) to $GITHUB_OUTPUT without the required sanitization. An attacker can inject newlines to poison GITHUB_OUTPUT.

Locations:

- `action.yaml:116`

### github-env-injection (severity: high)

The 'Gather additional files' step writes attacker-controlled input (inputs.additional_files via INPUT_ADDITIONAL_FILES) to $GITHUB_OUTPUT without the required sanitization. An attacker can inject newlines to poison GITHUB_OUTPUT.

Locations:

- `action.yaml:124`

### github-env-injection (severity: high)

The 'Run the check' step writes filepaths derived from attacker-controlled input (inputs.scandir via INPUT_SCANDIR) to $GITHUB_OUTPUT without the required sanitization. An attacker can inject newlines to poison GITHUB_OUTPUT.

Locations:

- `action.yaml:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 7 findings in action.yaml:

**script-injection (3 findings):**
- 'Download shellcheck' step: Added ACTION_PATH and RUNNER_OS env vars, replaced all ${{ github.action_path }} and ${{ runner.os }} template expressions in the shell script with ${ACTION_PATH} and ${RUNNER_OS}.
- 'Display shellcheck version' step: Added ACTION_PATH env var, replaced ${{ github.action_path }} with ${ACTION_PATH}.
- 'Run the check' step: Added ACTION_PATH env var, replaced both ${{ github.action_path }} occurrences with ${ACTION_PATH}.

**github-env-injection (4 findings):**
- 'Set options' step: Sanitized options array with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
- 'Gather excluded paths' step: Sanitized excludes array with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
- 'Gather additional files' step: Sanitized files array with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
- 'Run the check' step: Sanitized filepaths array with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. Also quoted $GITHUB_OUTPUT references throughout.

