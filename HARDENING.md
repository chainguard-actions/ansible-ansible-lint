<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.10.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Process inputs' writes `${{ inputs.working_directory }}` and `${{ github.workspace }}` straight to $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` cleanup first: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`. A newline in the input could add extra outputs.

Locations:

- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Determine github action ref' builds action_ref from inputs.override_version, inputs.gh_action_ref and github.action_ref (passed in through env vars), then writes it to $GITHUB_ENV without stripping newlines: `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. A newline in either input could set any environment variable for later steps.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Step 'Process inputs' breaks rule (a): it puts expressions directly in the script with `if [[ -n "${{ inputs.working_directory }}" ]]; then` and `echo "working_directory=${{ inputs.working_directory }}"` / `${{ github.workspace }}`.

Locations:

- `action.yml:48`

### script-injection (severity: high)

Step 'Generate .git/ansible-lint-requirements.txt' breaks rule (b): GH_ACTION_REF comes from inputs through env.ACTION_REF and is expanded without quotes in the URL. `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`

Locations:

- `action.yml:84`

### script-injection (severity: high)

Step 'Install ansible-lint' breaks rule (a): it puts expressions directly in the script with `if [[ "${{ inputs.setup_python }}" == "true" ]]` and `uv tool install --python ${{ inputs.python_version }} ...`.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Steps 'Install role and collection dependencies' and 'Run ansible-lint' break rule (a): `run: ansible-galaxy install -r ${{ inputs.requirements_file }}`, `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?`.

Locations:

- `action.yml:116`
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

**Fixes applied:** github-env-injection, script-injection, static-inline-injection

**Notes:**

All changes are in action.yml. Process inputs: working_directory and github.workspace are now passed in through env:, newlines are stripped, and the result is written once to "$GITHUB_OUTPUT". Determine github action ref: action_ref has newlines stripped before it is written to GITHUB_ENV. Generate reqs: the wget output path and URL are now quoted. Install ansible-lint: setup_python and python_version are passed in through env: and the version is quoted. ansible-galaxy: requirements_file is passed in through env: and quoted. Run ansible-lint: expected_return_code and args are passed in through env:. Upstream pasted args straight into the script, so the shell honored quotes in it. args is now split into a bash array with xargs, which honors quotes the same way. This split has an empty-value guard, an unmatched-quote check and xargs -r. No other behaviour changed. Nothing was run or tested.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, invalid-yaml

**Notes:**

Rebuilt the damaged 'Generate .git/ansible-lint-requirements.txt' and 'Install ansible-lint' steps. Generate step: put the working-directory key back, set reqs_file from git rev-parse --show-toplevel, stripped newlines before writing it to GITHUB_OUTPUT, and quoted the wget arguments. Removed the stray 'PUT' and the repeated lines. Install ansible-lint: setup_python and python_version now come in through env, and the two half-written uv lines are now one. Galaxy step: requirements_file now comes in through env and is quoted. Run ansible-lint: expected_return_code and args now come in through env. args is split into a bash array the same quote-aware way upstream's pasted-in expression was split (xargs, an unmatched-quote check, and an -n guard). The ${{ }} expressions in YAML comments were removed. Small edits kept coming out garbled, so the whole file was rewritten once and read back to check it. I didn't parse it with a YAML tool or run the action. Its run steps are now ordinary block scalars with no stray text.

