# ci-cd

Shared reusable GitHub Actions workflows for application repositories.

## Workflows

- `.github/workflows/python-ci.yml`
  Reusable Python quality, test, and Docker build checks.
- `.github/workflows/codeql.yml`
  Reusable CodeQL analysis workflow.
- `.github/workflows/version-check.yml`
  Reusable `VERSION` bump validation for pull requests.
- `.github/workflows/release-tag.yml`
  Reusable tag creation workflow based on the repository `VERSION` file.

## Example usage

```yaml
jobs:
  quality-and-tests:
    uses: LukaszRemkowicz/ci-cd/.github/workflows/python-ci.yml@main
    with:
      python_version: "3.13"
      dependency_install_command: uv sync --group dev
      pre_commit_command: uv run pre-commit run --all-files
      test_command: uv run pytest
      docker_compose_validate_command: docker compose config
      docker_build_command: docker compose build app tests
```
