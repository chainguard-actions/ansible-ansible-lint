<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.10.1** was hardened automatically. 14 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Process inputs' writes ${{ inputs.working_directory }} (and ${{ github.workspace }}) straight to $GITHUB_OUTPUT with no printf '%s' | tr -d '\n\r' cleanup first: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`. A newline in the value can inject extra outputs.

Locations:

- `action.yml:49`

### github-env-injection (severity: high)

Step 'Determine github action ref' builds action_ref from inputs.override_version, inputs.gh_action_ref or github.action_ref (passed in through env) and writes it with `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. Newlines are not stripped first, so arbitrary environment variables can be injected.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Step 'Process inputs' breaks rule (a): an expression is put straight into the script: `if [[ -n "${{ inputs.working_directory }}" ]]; then echo "working_directory=${{ inputs.working_directory }}" ...` and `echo "working_directory=${{ github.workspace }}"`.

Locations:

- `action.yml:48`

### script-injection (severity: high)

Step 'Generate .git/ansible-lint-requirements.txt' breaks rule (b): the untrusted GH_ACTION_REF (from inputs or github.action_ref) is expanded without quotes: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`.

Locations:

- `action.yml:84`

### script-injection (severity: high)

Step 'Install ansible-lint' breaks rule (a): `if [[ "${{ inputs.setup_python }}" == "true" ]]` and `uv tool install --python ${{ inputs.python_version }} ...` put expressions straight into the script.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Step 'Install role and collection dependencies' breaks rule (a): `run: ansible-galaxy install -r ${{ inputs.requirements_file }}`.

Locations:

- `action.yml:116`

### script-injection (severity: high)

Step 'Run ansible-lint' breaks rule (a): `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?` put expressions straight into the script.

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

**Fixes applied:** github-env-injection, script-injection, static-inline-injection

**Notes:**

All changes are in action.yml:
- **Process inputs:** working_directory and github.workspace now come in through env:. Newlines are stripped before the value is written to GITHUB_OUTPUT.
- **Determine ref:** newlines are stripped from action_ref before it is written to GITHUB_ENV. github.action_ref would name a commit of the mirror, not of ansible/ansible-lint, so the fallback ref used to fetch upstream code is now the literal upstream_ref v26.10.1. override_version and gh_action_ref still take precedence over it.
- **wget step:** the output file and the URL are now quoted.
- **Install step:** setup_python and python_version moved to env:, and python_version is quoted. I also removed two old comment lines that contained github.action_ref expressions.
- **Galaxy step:** requirements_file moved to env: and is quoted.
- **Run step:** expected_return_code and args moved to env:. Upstream pasted args straight into the script, so the shell honoured quotes in it. To keep that, args is split with the quote-aware xargs form: a check that fails on an unmatched quote, then the shell's built-in printf run through sh -c, so a lone --version still reaches ansible-lint. An empty args passes no arguments.

### Iteration 2

**Fixes applied:** script-injection, invalid-yaml

**Notes:**

Rebuilt the damaged 'Set up uv' step. It is now a plain astral-sh/setup-uv step, still pinned to its SHA, and a separate 'Install ansible-lint' run step follows it. The new step uses shell: bash and an env: block holding GH_ACTION_REF, INPUT_SETUP_PYTHON and INPUT_PYTHON_VERSION. I removed the duplicated broken uv line and replaced the inline ${{ inputs.python_version }} with "$INPUT_PYTHON_VERSION". The broken edit had removed whatever setup-uv originally had under with:, so I left with: off. uv gets the Python version directly through --python.

