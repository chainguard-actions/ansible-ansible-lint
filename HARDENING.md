<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.11.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.11.1** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Process inputs' breaks rule (a): expressions are placed straight into the run script. Offending lines: `if [[ -n "${{ inputs.working_directory }}" ]]; then`, `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` and `echo "working_directory=${{ github.workspace }}" >> $GITHUB_OUTPUT`. Fix: pass these values through env: variables and quote them.

Locations:

- `action.yml:48`
- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Process inputs' writes inputs.working_directory (and github.workspace) to $GITHUB_OUTPUT without first running printf '%s' ... | tr -d '\n\r'. A newline in the value can inject extra outputs: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Determine github action ref' builds action_ref from the untrusted inputs override_version and gh_action_ref (and github.action_ref), all routed through env vars. It then writes the value to $GITHUB_ENV without first running printf '%s' ... | tr -d '\n\r': `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. A newline in either input can inject arbitrary environment variables, such as LD_PRELOAD or PATH, into later steps.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Step 'Generate .git/ansible-lint-requirements.txt' breaks rule (b): GH_ACTION_REF holds env.ACTION_REF, which comes from user inputs, and it is expanded unquoted inside a URL: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`. Fix: quote the URL (and $reqs_file).

Locations:

- `action.yml:85`

### script-injection (severity: high)

Step 'Install ansible-lint' breaks rule (a): `if [[ "${{ inputs.setup_python }}" == "true" ]]; then` and `uv tool install --python ${{ inputs.python_version }} --with-executables-from ansible-core ...` put input expressions straight into the run script.

Locations:

- `action.yml:103`
- `action.yml:105`

### script-injection (severity: high)

Step 'Install role and collection dependencies from requirements file' breaks rule (a): `run: ansible-galaxy install -r ${{ inputs.requirements_file }}` puts the input expression straight into the shell command.

Locations:

- `action.yml:116`

### script-injection (severity: high)

Step 'Run ansible-lint' breaks rule (a): `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?` put input expressions straight into the run script.

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

Hardened action.yml: every ${{ }} expression in a run: script now goes through env: and is used as a quoted variable. Before anything is written to GITHUB_OUTPUT or GITHUB_ENV (working_directory, ACTION_REF, reqs_file), newlines are stripped with printf | tr -d '\n\r'. In the wget step, both the output path and the URL are now quoted. Upstream pasted inputs.args into the script directly, so it is now split into an array the same way the shell did (quote-aware xargs, with the empty-value guard and the unmatched-quote check), keeping multiple arguments working. requirements_file, python_version and expected_return_code are single values and are passed quoted. The default ref used to fetch upstream's own code (requirements-lock.txt and the git+https install) now falls back to the upstream tag 'v26.11.1' instead of github.action_ref, which would name a ref in the mirror. The override_version and gh_action_ref inputs still take precedence. I only edited the file; nothing was run, so the action and its tests haven't been tried after these changes.

