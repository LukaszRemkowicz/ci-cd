# ci-cd

Shared reusable GitHub Actions workflows for application repositories.

## Workflows

- `.github/workflows/python-ci.yml`
  Reusable Python quality, test, and Docker build checks for simple service repos.
- `.github/workflows/python-uv-ci.yml`
  Reusable Python/uv quality and test workflow with optional Postgres.
- `.github/workflows/node-ci.yml`
  Reusable Node quality and test workflow for frontend projects.
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

## Notes

- Keep project-specific environment values in the application repository.
- Prefer thin workflow wrappers in application repos and reusable logic here.
- Keep project-specific release and deployment logic in the application repo unless the release contract is truly shared.
