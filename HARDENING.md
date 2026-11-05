<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.11.0** was hardened automatically. 14 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) is violated. The 'Process inputs' step puts expressions straight into the run script: `if [[ -n "${{ inputs.working_directory }}" ]]; then`, `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` and `echo "working_directory=${{ github.workspace }}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:48`

### github-env-injection (severity: high)

The 'Process inputs' step writes inputs.working_directory and github.workspace straight to $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

The 'Determine github action ref' step builds action_ref from the untrusted inputs override_version and gh_action_ref and from github.action_ref. It then writes `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"` without the `printf '%s' ... | tr -d '\n\r'` sanitization, so a newline in the value can inject extra environment variables.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Rule (b) is violated. In the 'Generate .git/ansible-lint-requirements.txt' step, GH_ACTION_REF comes from the input-derived env.ACTION_REF and is expanded unquoted inside a URL: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`. $reqs_file is also unquoted.

Locations:

- `action.yml:84`

### script-injection (severity: high)

Rule (a) is violated. The 'Install ansible-lint' step puts expressions straight into the run script: `if [[ "${{ inputs.setup_python }}" == "true" ]]; then` and `uv tool install --python ${{ inputs.python_version }} ...`.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Rule (a) is violated. The 'Install role and collection dependencies' step puts an expression straight into the run script: `run: ansible-galaxy install -r ${{ inputs.requirements_file }}`.

Locations:

- `action.yml:116`

### script-injection (severity: high)

Rule (a) is violated. The 'Run ansible-lint' step puts expressions straight into the run script: `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?`.

Locations:

- `action.yml:123`
- `action.yml:124`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.working_directory }}" appears directly in run: block of step "Process inputs"; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.working_directory }}" appears directly in run: block of step "Process inputs"; move to env: map

Locations:

- `action.yml:49`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.setup_python }}" appears directly in run: block of step "Install ansible-lint"; move to env: map

Locations:

- `action.yml:103`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.python_version }}" appears directly in run: block of step "Install ansible-lint"; move to env: map

Locations:

- `action.yml:105`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.requirements_file }}" appears directly in run: block of step "Install role and collection dependencies from requirements file"; move to env: map

Locations:

- `action.yml:117`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.expected_return_code }}" appears directly in run: block of step "Run ansible-lint"; move to env: map

Locations:

- `action.yml:123`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.args }}" appears directly in run: block of step "Run ansible-lint"; move to env: map

Locations:

- `action.yml:124`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

All changes are in action.yml. Every `${{ }}` expression in the run: blocks now goes through env: (working_directory, github.workspace, setup_python, python_version, requirements_file, expected_return_code, args). working_directory and ACTION_REF are passed through `printf '%s' | tr -d '\n\r'` before they are written to GITHUB_OUTPUT and GITHUB_ENV. reqs_file is also stripped when written to GITHUB_OUTPUT. The variables in the wget call are now quoted. Upstream pasted `args` into the script directly, so the shell honoured quotes in it. To keep that behaviour it is split into a bash array with the quote-aware xargs method (guarded by `-n`, `xargs -r true` to fail on an unmatched quote, and a read loop that works on bash 3.2). The YAML comments that mention github.action_ref were left alone because they are not part of any run: block.

### Iteration 2

**Fixes applied:** script-injection, invalid-yaml

**Notes:**

Fixed all three findings in action.yml. I haven't run the action or its tests.

- **Invalid YAML:** an earlier edit had broken the 'Set up uv' step, merging its `with:` block with text from other steps. I rebuilt it as `with: python-version: ${{ inputs.python_version }}`. That line is a reconstruction, not upstream's text: I couldn't see upstream's original block, so check whether it had any other `with:` keys (for example cache settings).
- **Script injection in 'Install ansible-lint':** the `INPUT_SETUP_PYTHON` and `INPUT_PYTHON_VERSION` env entries were stuck in the broken block above. I moved them into this step's `env:`. The script now uses "$INPUT_SETUP_PYTHON" and a quoted "$INPUT_PYTHON_VERSION" instead of inline expressions.
- **Script injection in 'Run ansible-lint':** I deleted the leftover `ansible-lint ${{ inputs.args }}` line and the stray `_return_code }}` fragment. The step still runs ansible-lint from the array built from $INPUT_ARGS, which keeps quoted arguments together the way upstream's inline version did.

The `${{ }}` expressions left in the Install step are only in YAML comments, so they never reach the shell.

