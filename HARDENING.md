<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.11.0** was hardened automatically. 13 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Process inputs' writes inputs.working_directory and github.workspace straight into $GITHUB_OUTPUT with no printf '%s' | tr -d '\n\r' cleanup first: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`. A newline in the input could inject extra outputs.

Locations:

- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Determine github action ref' writes action_ref, which comes from inputs.override_version, inputs.gh_action_ref or github.action_ref, into $GITHUB_ENV without removing newlines first: `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. An attacker could set arbitrary environment variables.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Step 'Generate .git/ansible-lint-requirements.txt' breaks rule (b): $GH_ACTION_REF holds env.ACTION_REF, which the caller controls, and it is expanded without quotes inside the wget URL: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`.

Locations:

- `action.yml:84`

### script-injection (severity: high)

Step 'Install ansible-lint' breaks rule (a): expressions are put straight into the run script: `if [[ "${{ inputs.setup_python }}" == "true" ]]` and `uv tool install --python ${{ inputs.python_version }} ...`.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Step 'Install role and collection dependencies from requirements file' breaks rule (a): `run: ansible-galaxy install -r ${{ inputs.requirements_file }}` puts the input straight into the shell command.

Locations:

- `action.yml:115`

### script-injection (severity: high)

Step 'Run ansible-lint' breaks rule (a): `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?` put input expressions straight into the shell script.

Locations:

- `action.yml:122`
- `action.yml:123`

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

action.yml: Process inputs now reads working_directory and github.workspace from env and strips newlines before writing to GITHUB_OUTPUT. Determine github action ref strips newlines before writing ACTION_REF to GITHUB_ENV. Its fallback is now the literal upstream_ref v26.11.0 in place of github.action_ref, which would name a mirror commit; the ref feeds the raw.githubusercontent and pip/uv fetches from upstream. The wget URL and output path are quoted. setup_python, python_version, requirements_file, expected_return_code and args are moved to env vars. args uses the quote-aware split: a guarded xargs -r split through sh -c printf into a bash array, with the xargs -r true check for unmatched quotes. This matches upstream's inline expansion. Removed the ${{ github.action_ref }} expression comments from the Install step.

