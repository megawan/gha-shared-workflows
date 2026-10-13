# gha-shared-workflows

A collection of reusable GitHub Actions workflows meant to be called from other repositories.

Reference them by version tag (`@v1`), not `@main`, so that a change here does not break every repo at once.

## Layout

```
.github/workflows/   reusable workflows, grouped by filename prefix
examples/            ready-to-copy caller workflows, one folder per group
```

GitHub only loads workflows from the top level of `.github/workflows/`, so workflow files cannot be placed in subfolders. They are grouped by prefix instead: `automation-`, `ci-`, `security-`, `site-` and `terraform-`.

## Workflows

### Automation (`automation-*`)

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`automation-pr-title-lint.yml`](.github/workflows/automation-pr-title-lint.yml) | Validate PR titles (Conventional Commits) | `pull_request_target` |
| [`automation-labeler.yml`](.github/workflows/automation-labeler.yml) | Label PRs automatically by changed paths | `pull_request_target` |
| [`automation-stale.yml`](.github/workflows/automation-stale.yml) | Mark and close inactive issues and PRs | `schedule`, `workflow_dispatch` |
| [`automation-auto-merge-dependabot.yml`](.github/workflows/automation-auto-merge-dependabot.yml) | Approve and auto-merge Dependabot PRs | `pull_request` |
| [`automation-sync-labels.yml`](.github/workflows/automation-sync-labels.yml) | Sync repo labels with a definition file | `push`, `workflow_dispatch` |
| [`automation-actionlint.yml`](.github/workflows/automation-actionlint.yml) | Lint workflow files with actionlint | `pull_request`, `push` |

Examples: [`examples/automation/`](examples/automation)

### CI (`ci-*`)

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`ci-node.yml`](.github/workflows/ci-node.yml) | Install, lint, test and build with npm, pnpm or yarn | `push`, `pull_request` |
| [`ci-python.yml`](.github/workflows/ci-python.yml) | Install, optional lint, and test with pytest | `push`, `pull_request` |
| [`ci-go.yml`](.github/workflows/ci-go.yml) | `go vet`, golangci-lint, `go test -race`, `go build` | `push`, `pull_request` |
| [`ci-docker-build-push.yml`](.github/workflows/ci-docker-build-push.yml) | Build and push (multi-arch) images with metadata tags and cache | `push`, `pull_request`, tags |

Examples: [`examples/ci/`](examples/ci)

### Security (`security-*`)

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`security-codeql.yml`](.github/workflows/security-codeql.yml) | CodeQL static analysis per language, results in code scanning | `push`, `pull_request`, `schedule` |
| [`security-dependency-review.yml`](.github/workflows/security-dependency-review.yml) | Block vulnerable or disallowed dependencies and licenses | `pull_request` |
| [`security-secret-scan.yml`](.github/workflows/security-secret-scan.yml) | Detect leaked secrets with gitleaks (redacted output) | `push`, `pull_request` |

Examples: [`examples/security/`](examples/security)

### Site (`site-*`)

Checks for static sites (for example GitHub Pages) that have no build step.

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`site-html-validate.yml`](.github/workflows/site-html-validate.yml) | Validate HTML syntax, structure and accessibility with html-validate | `pull_request`, `push` |
| [`site-link-check.yml`](.github/workflows/site-link-check.yml) | Check links and asset references (images, scripts, audio) with lychee | `pull_request`, `push`, `schedule` |

Examples: [`examples/site/`](examples/site)

### Terraform (`terraform-*`)

Split into small workflows with separate permissions and OIDC authentication. The flow and security model are described in [`examples/terraform/README.md`](examples/terraform/README.md).

| Workflow | Purpose |
| --- | --- |
| [`terraform-validate.yml`](.github/workflows/terraform-validate.yml) | fmt, validate, TFLint (no credentials) |
| [`terraform-security.yml`](.github/workflows/terraform-security.yml) | Trivy misconfiguration scan |
| [`terraform-plan.yml`](.github/workflows/terraform-plan.yml) | Plan with PR comment |
| [`terraform-apply.yml`](.github/workflows/terraform-apply.yml) | Gated apply of a verified saved plan |
| [`terraform-drift.yml`](.github/workflows/terraform-drift.yml) | Scheduled drift detection with issue reporting |
| [`terraform-destroy.yml`](.github/workflows/terraform-destroy.yml) | Confirmed, gated destroy |

Examples: [`examples/terraform/`](examples/terraform)

## Calling a workflow

```yaml
jobs:
  ci:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/ci-node.yml@v1
    with:
      package-manager: pnpm
```

## Pull request template

PRs use [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md). Usage, reuse in other repositories, and filled-in examples are in [`examples/pull-request/`](examples/pull-request).

## Notes
- The caller must grant at least the `permissions` that the called workflow's job needs (the examples already do).
- This repo must be public, or private with *Settings > Actions > General > Access* allowing other repos to use it.
- This repository lints its own workflows with [`lint.yml`](.github/workflows/lint.yml), enforces PR titles with [`pr-title.yml`](.github/workflows/pr-title.yml), and keeps actions up to date with [Dependabot](.github/dependabot.yml).
