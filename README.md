# gha-shared-workflows

A collection of reusable GitHub Actions workflows meant to be called from other repositories.

Reference them by version tag (`@v1`), not `@main`, so that a change here does not break every repo at once.

## Layout

GitHub only loads workflows from `.github/workflows/` and does not support subfolders, so workflow files stay flat there and are grouped by filename prefix. Each group has a folder with its own documentation and examples.

| Group | Prefix | Docs and examples | Workflows |
| --- | --- | --- | --- |
| PR and repo automation | `automation-` | [`automation/`](automation/README.md) | PR title lint, labeler, stale, Dependabot auto-merge, label sync |
| CI | `ci-` | [`ci/`](ci/README.md) | Node, Python, Go, Docker build and push |
| Terraform | `terraform-` | [`terraform/`](terraform/README.md) | validate, security scan, plan, apply, drift, destroy |

```
.github/workflows/
  automation-*.yml
  ci-*.yml
  terraform-*.yml
automation/   README.md
ci/           README.md
terraform/    README.md, examples/
```

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

## Notes
- The caller must grant at least the `permissions` that the called workflow's job needs (the examples above already do).
- This repo must be public, or private with *Settings > Actions > General > Access* allowing other repos to use it.
