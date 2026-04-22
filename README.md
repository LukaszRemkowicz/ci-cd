# ci-cd

Shared reusable GitHub Actions workflows for application repositories.

## Workflows

- `.github/workflows/python-quality-uv.yml`
  Reusable Python quality workflow with explicit formatter, lint, and type-check commands.
- `.github/workflows/python-tests-uv.yml`
  Reusable Python test workflow with optional Postgres.
- `.github/workflows/node-quality.yml`
  Reusable Node quality workflow for frontend projects.
- `.github/workflows/node-tests.yml`
  Reusable Node test workflow for frontend projects.
- `.github/workflows/codeql.yml`
  Reusable CodeQL analysis workflow with multi-language and optional config file support.
- `.github/workflows/trivy-scan.yml`
  Reusable Trivy SARIF scan workflow.
- `.github/workflows/version-check.yml`
  Reusable `VERSION` bump validation for pull requests.
- `.github/workflows/release-tag.yml`
  Reusable tag creation workflow based on the repository `VERSION` file.
- `.github/workflows/branch-protection.yml`
  Reusable branch protection rules for pull requests.
- `.github/workflows/pr-status.yml`
  Reusable pull request status comment workflow.

## Workflow Contracts

The shared workflows are called from application repositories with:

```yaml
jobs:
  some-job:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/<workflow>.yml@main
    with:
      ...
```

The sections below describe the supported `with:` inputs for each reusable
workflow.

### `python-quality-uv.yml`

Use for Python quality checks with explicit formatter, lint, and type-check
commands instead of pre-commit.

The workflow installs project dependencies and runs the shared quality commands
through `uv run`.

Inputs:
- `python_version`
  Python version for the runner.
- `working_directory`
  Directory where the commands should run.

The shared workflow runs these quality commands:
- `uv run isort . --settings-path pyproject.toml --profile black --check-only`
- `uv run black . --config pyproject.toml --check`
- `uv run flake8 . --exclude migrations --toml-config pyproject.toml`
- `uv run ruff check . --config pyproject.toml`
- `uv run mypy .`

Example:

```yaml
jobs:
  backend-quality:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/python-quality-uv.yml@main
    with:
      working_directory: backend
```

### `python-tests-uv.yml`

Use for Python projects that run through `uv` and may optionally require a
Postgres service in CI.

The workflow installs project dependencies and then runs the configured test
command.

Inputs:
- `python_version`
  Python version for the runner.
- `working_directory`
  Directory where the commands should run.
- `postgres_enabled`
  Set `true` when the project requires Postgres in CI.
- `postgres_image`
  Postgres image to run when enabled.
- `postgres_db`
  Database name for the Postgres service.
- `postgres_user`
  Postgres user for the service.
- `postgres_password`
  Postgres password for the service.
- `postgres_port`
  Host port to expose Postgres on.

Important:
- project-specific env stays in the caller repo
- secrets stay in the caller repo
- this shared workflow provides mechanics, not project configuration

Example:

```yaml
jobs:
  backend-tests:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/python-tests-uv.yml@main
    with:
      working_directory: backend
      postgres_enabled: true
      postgres_db: test_db
      postgres_user: postgres
      postgres_password: postgres
      postgres_port: "5432"
    secrets: inherit
```

### `node-quality.yml`

Use for frontend or other Node-based quality checks.

Inputs:
- `node_version`
  Node.js version for the runner.
- `working_directory`
  Directory where Node commands should run.
- `cache_dependency_path`
  Dependency file path used for npm cache invalidation.
- `install_command`
  Command used to install dependencies.
- `lint_command`
  Command used to run linting.
- `format_check_command`
  Command used to run formatting checks.
- `type_check_command`
  Command used to run type checks.

Example:

```yaml
jobs:
  frontend-quality:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/node-quality.yml@main
    with:
      node_version: "24"
      working_directory: frontend
      cache_dependency_path: frontend/package-lock.json
      install_command: npm ci
      lint_command: npm run lint
      format_check_command: npm run format:check
      type_check_command: npx tsc --noEmit
```

### `node-tests.yml`

Use for frontend or other Node-based test checks.

Inputs:
- `node_version`
  Node.js version for the runner.
- `working_directory`
  Directory where Node commands should run.
- `cache_dependency_path`
  Dependency file path used for npm cache invalidation.
- `install_command`
  Command used to install dependencies.
- `test_command`
  Command used to run tests.
- `test_environment`
  Optional multiline `KEY=VALUE` pairs exported before the test command runs.

### `codeql.yml`

Use for CodeQL analysis.

Inputs:
- `languages_json`
  JSON array of languages, for example `["python"]` or
  `["javascript","python"]`.
- `config_file`
  Optional CodeQL config file path in the caller repo.

Examples:

```yaml
jobs:
  analyze:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/codeql.yml@main
    with:
      languages_json: '["python"]'
```

```yaml
jobs:
  analyze:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/codeql.yml@main
    with:
      languages_json: '["javascript","python"]'
      config_file: .github/codeql-config.yml
```

### `trivy-scan.yml`

Use for reusable Trivy filesystem scanning with SARIF upload.

Inputs:
- `scan_path`
  Path to scan.
- `severity`
  Comma-separated severities to include.
- `output_file`
  SARIF output file name.
- `category`
  SARIF category label.
- `trivy_image`
  Trivy container image to use.

### `version-check.yml`

Use for `VERSION` validation on pull requests.

Inputs:
- `version_file`
  Version file path.
- `base_branch`
  Base branch that should trigger the check.
- `head_branch`
  Head branch that should trigger the check.

### `release-tag.yml`

Use for creating release tags from the repository `VERSION` file after changes
land on `main`.

Inputs:
- `version_file`
  Version file path.
- `tag_prefix`
  Tag prefix, usually `v`.

### `branch-protection.yml`

Use for branch-source policy on pull requests.

Inputs:
- `target_branch`
  Protected base branch.
- `allowed_exact_branches`
  Comma-separated exact branch names allowed to merge into the target branch.
- `allowed_pattern_branches`
  Comma-separated regex patterns allowed to merge into the target branch.

### `pr-status.yml`

Use for posting or updating a PR summary comment.

Inputs:
- `status_items_json`
  JSON object mapping labels to job results.
- `success_message`
  Message appended when all items are successful.
- `failure_message`
  Message appended when any item fails.

Example:

```yaml
jobs:
  pr-status:
    needs: [quality-and-tests]
    if: always()
    uses: LukaszRemkowicz/ci-cd/.github/workflows/pr-status.yml@main
    with:
      status_items_json: >-
        {"CI (quality, tests, docker build)":"${{ needs.quality-and-tests.result }}"}
      success_message: All service CI checks passed.
      failure_message: Service CI failed. Please review the workflow logs.
```

## Notes

- Keep project-specific environment values in the application repository.
- Prefer thin workflow wrappers in application repos and reusable logic here.
- Keep project-specific release and deployment logic in the application repo unless the release contract is truly shared.
