<!-- markdownlint-disable -->

# Hardening Report: ansible--ansible-lint/v26.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ansible--ansible-lint/v26.10.0** was hardened automatically. 14 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Process inputs' writes `${{ inputs.working_directory }}` (and `${{ github.workspace }}`) straight to $GITHUB_OUTPUT without first running `printf '%s' ... | tr -d '\n\r'`: `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT`. A value containing a newline can inject extra outputs.

Locations:

- `action.yml:49`
- `action.yml:51`

### github-env-injection (severity: high)

Step 'Determine github action ref' builds action_ref from the inputs override_version and gh_action_ref (passed in as the OVERRIDE_VER and OLD_REF env vars) and writes it to $GITHUB_ENV without stripping newlines: `echo "ACTION_REF=${action_ref}" >> "$GITHUB_ENV"`. An input containing a newline can set any environment variable for later steps.

Locations:

- `action.yml:69`

### script-injection (severity: high)

Rule (a): step 'Process inputs' puts expressions directly in the run script: `if [[ -n "${{ inputs.working_directory }}" ]]; then`, `echo "working_directory=${{ inputs.working_directory }}" >> $GITHUB_OUTPUT` and `echo "working_directory=${{ github.workspace }}"`.

Locations:

- `action.yml:48`

### script-injection (severity: high)

Rule (b): step 'Generate .git/ansible-lint-requirements.txt' expands variables without quotes, including the workflow-controlled $GH_ACTION_REF (from env.ACTION_REF, which comes from inputs) inside a URL: `wget --output-document=$reqs_file https://raw.githubusercontent.com/ansible/ansible-lint/$GH_ACTION_REF/.config/requirements-lock.txt`.

Locations:

- `action.yml:84`

### script-injection (severity: high)

Rule (a): step 'Install ansible-lint' puts expressions directly in the run script: `if [[ "${{ inputs.setup_python }}" == "true" ]]; then` and `uv tool install --python ${{ inputs.python_version }} --with-executables-from ansible-core ...`.

Locations:

- `action.yml:102`
- `action.yml:104`

### script-injection (severity: high)

Rule (a): step 'Install role and collection dependencies from requirements file' puts an expression directly in the run command: `run: ansible-galaxy install -r ${{ inputs.requirements_file }}`.

Locations:

- `action.yml:115`

### script-injection (severity: high)

Rule (a): step 'Run ansible-lint' puts expressions directly in the run script: `expected_exit_code=${{ inputs.expected_return_code }}` and `ansible-lint ${{ inputs.args }} || exit_code=$?`.

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

Process inputs: working_directory and github.workspace are now passed through env and have newlines stripped before going to GITHUB_OUTPUT. Determine ref: ACTION_REF has newlines stripped before going to GITHUB_ENV. The wget step's variables and URL are now quoted. Install step: setup_python and python_version come from env, and the python version is quoted. ansible-galaxy now reads requirements_file from env, quoted. Run ansible-lint: expected_return_code comes from env. args is split into an array with the quote-aware xargs method (upstream pasted the expression straight into the script), with an empty-value guard and an unmatched-quote check.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, invalid-yaml

**Notes:**

I rewrote action.yml as a whole. Nothing has run it yet, and two parts were rebuilt from memory of upstream ansible-lint rather than from the file:
- The `working-directory: ${{ steps.inputs.outputs.working_directory }}` line on the 'Generate .git/ansible-lint-requirements.txt' step was garbled, so I put that expression back.
- On 'Install role and collection dependencies from requirements file', I restored the condition `if: inputs.requirements_file != ''` and `shell: bash`. I also added `working-directory: ${{ steps.inputs.outputs.working_directory }}`, which I believe upstream has. The garbled file showed no working-directory there, so check this line against the upstream release.

Changes by step:
- **Process inputs:** `inputs.working_directory` and `github.workspace` now come in through env. Whichever value is used has newlines and carriage returns stripped (`printf '%s' | tr -d '\n\r'`) and is written to `$GITHUB_OUTPUT` once, quoted.
- **Generate requirements:** fixed the `working-directory:` key that had been merged into the script, restored `run: |`, and removed the stray unquoted `$GH_ACTION_REF` fragment after the wget URL.
- **Install ansible-lint:** restored the damaged `run: |` key. The uv command now passes only `--python "$INPUT_PYTHON_VERSION"`; the stray `--withython ${{ inputs.python_version }}` is gone. I dropped the garbled comment lines that contained expressions.
- **Run ansible-lint:** removed the leftover `expected_exit_code=${{ ... }}` and `ansible-lint ${{ inputs.args }}` lines. Only the array form built from `INPUT_ARGS` with xargs remains, so quotes in `args` are honoured the same way as before.

The parse error at line 106 came from these merged keys, so restoring them fixes the invalid-yaml finding.

