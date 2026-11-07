<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.10.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Process inputs' writes untrusted values to $GITHUB_OUTPUT without sanitizing them first: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` (and `${{ github.workspace }}` on line 51). A newline in the input can inject extra outputs. Fix: pass the value through env and run it through `printf '%s' | tr -d '\n\r'` before the write.

Locations:

- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Determine github action ref' writes action_ref to $GITHUB_ENV without sanitizing it: `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. action_ref comes from inputs.override_version, inputs.gh_action_ref and github.action_ref. A newline in it can inject arbitrary environment variables (e.g. LD_PRELOAD, BASH_ENV). Fix: use `printf '%s' "$action_ref" | tr -d '\n\r'` first.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Rule (a): step 'Process inputs' puts expressions directly into the run script: `if [[ -n "${{ inputs.working_directory }}" ]]; then`, `echo "working_directory=${{ inputs.working_directory }}"` and `echo "working_directory=${{ github.workspace }}"`.

Locations:

- `action.yml:48`

### script-injection (severity: high)

Rule (b): step 'Generate .git/ansible-lint-requirements.txt' expands variables without quotes: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`. GH_ACTION_REF comes from env.ACTION_REF, which is built from inputs, and it is unquoted inside the URL.

Locations:

- `action.yml:84`

### script-injection (severity: high)

Rule (a): step 'Install ansible-lint' puts expressions directly into the run script: `if [[ "${{ inputs.setup_python }}" == "true" ]]; then` and `uv tool install --python ${{ inputs.python_version }} ...`.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Rule (a): step 'Install role and collection dependencies from requirements file' puts an expression directly into the run script: `run: ansible-galaxy install -r ${{ inputs.requirements_file }}`.

Locations:

- `action.yml:115`

### script-injection (severity: high)

Rule (a): step 'Run ansible-lint' puts expressions directly into the run script: `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?`.

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

Changes to action.yml:
- Process inputs: working_directory and github.workspace now come in through env:. The value has newlines stripped (tr -d '\n\r') before it is written to GITHUB_OUTPUT.
- Determine github action ref: action_ref has newlines stripped before it is written to GITHUB_ENV. The default ref no longer comes from github.action_ref, which is a commit or tag of the mirror. It is now the upstream release v26.10.0, because it decides which ansible-lint gets downloaded from upstream.
- Generate reqs: reqs_file and the wget URL are quoted, and reqs_file is stripped of newlines before it is written to the output.
- Install ansible-lint: setup_python and python_version come in through env: and are quoted.
- requirements_file comes in through env: and is quoted as a single path.
- Run ansible-lint: expected_return_code and args come in through env:. Upstream pasted args straight into the script, so the shell honoured quotes in it. To keep that, args is split with the quote-aware xargs method into a bash array, with a guard for an empty value, a check that fails on an unmatched quote, and the shell's built-in printf so that '--version' still reaches ansible-lint.

