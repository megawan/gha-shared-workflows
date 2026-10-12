# gha-shared-workflows

A collection of reusable GitHub Actions workflows meant to be called from other repositories.

Reference them by version tag (`@v1`), not `@main`, so that a change here does not break every repo at once.

## PR / repo automation

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`pr-title-lint.yml`](.github/workflows/pr-title-lint.yml) | Validate PR titles (Conventional Commits) | `pull_request_target` |
| [`labeler.yml`](.github/workflows/labeler.yml) | Label PRs automatically by changed paths | `pull_request_target` |
| [`stale.yml`](.github/workflows/stale.yml) | Mark and close inactive issues and PRs | `schedule`, `workflow_dispatch` |
| [`auto-merge-dependabot.yml`](.github/workflows/auto-merge-dependabot.yml) | Approve and auto-merge Dependabot PRs | `pull_request` |
| [`sync-labels.yml`](.github/workflows/sync-labels.yml) | Sync repo labels with a definition file | `push`, `workflow_dispatch` |

## CI

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`node-ci.yml`](.github/workflows/node-ci.yml) | Install, lint, test and build with npm, pnpm or yarn | `push`, `pull_request` |
| [`python-ci.yml`](.github/workflows/python-ci.yml) | Install, optional lint, and test with pytest | `push`, `pull_request` |
| [`go-ci.yml`](.github/workflows/go-ci.yml) | `go vet`, golangci-lint, `go test -race`, `go build` | `push`, `pull_request` |
| [`docker-build-push.yml`](.github/workflows/docker-build-push.yml) | Build and push (multi-arch) images with metadata tags and cache | `push`, `pull_request`, tags |
| [`terraform-plan-apply.yml`](.github/workflows/terraform-plan-apply.yml) | fmt, validate, plan, and optional gated apply | `pull_request`, `push` |

## PR / repo automation usage examples

### PR title lint
```yaml
name: PR Title
on:
  pull_request_target:
    types: [opened, edited, synchronize, reopened]
jobs:
  lint:
    permissions:
      pull-requests: read
    uses: OWNER/gha-shared-workflows/.github/workflows/pr-title-lint.yml@v1
    with:
      require-scope: false
```

### Labeler
Requires a `.github/labeler.yml` file in the caller repo.
```yaml
name: Labeler
on: pull_request_target
jobs:
  label:
    permissions:
      contents: read
      pull-requests: write
    uses: OWNER/gha-shared-workflows/.github/workflows/labeler.yml@v1
```

### Stale
```yaml
name: Stale
on:
  schedule:
    - cron: "0 3 * * *"
  workflow_dispatch:
jobs:
  stale:
    permissions:
      issues: write
      pull-requests: write
    uses: OWNER/gha-shared-workflows/.github/workflows/stale.yml@v1
    with:
      days-before-stale: 30
```

### Auto-merge Dependabot
Enable "Allow auto-merge" and required status checks in the caller repo.
```yaml
name: Dependabot
on: pull_request
jobs:
  auto-merge:
    permissions:
      contents: write
      pull-requests: write
    uses: OWNER/gha-shared-workflows/.github/workflows/auto-merge-dependabot.yml@v1
    with:
      allowed-update-types: patch,minor
```

### Sync labels
Requires a `.github/labels.yml` file (`crazy-max/ghaction-github-labeler` format) in the caller repo.
```yaml
name: Sync Labels
on:
  push:
    branches: [main]
    paths: [.github/labels.yml]
  workflow_dispatch:
jobs:
  sync:
    permissions:
      contents: read
      issues: write
    uses: OWNER/gha-shared-workflows/.github/workflows/sync-labels.yml@v1
```

## CI usage examples

### Node CI
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
jobs:
  ci:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/node-ci.yml@v1
    with:
      package-manager: pnpm
      scripts: lint,test,build
```

### Python CI
```yaml
jobs:
  ci:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/python-ci.yml@v1
    with:
      python-version: "3.13"
      lint-command: ruff check .
```

### Go CI
```yaml
jobs:
  ci:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/go-ci.yml@v1
```

### Docker build and push
Pushes to GHCR with the job token. Images are built but not pushed on pull requests.
```yaml
on:
  push:
    branches: [main]
    tags: ["v*"]
  pull_request:
jobs:
  image:
    permissions:
      contents: read
      packages: write
    uses: OWNER/gha-shared-workflows/.github/workflows/docker-build-push.yml@v1
    with:
      platforms: linux/amd64,linux/arm64
```
For other registries, set `registry` and pass the `registry-username` and `registry-password` secrets.

### Terraform plan and apply
Plan on pull requests; plan and apply on pushes to `main`, gated by an environment.
```yaml
on:
  pull_request:
  push:
    branches: [main]
jobs:
  terraform:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/terraform-plan-apply.yml@v1
    with:
      working-directory: infra
      apply: ${{ github.event_name == 'push' }}
      environment: production
    secrets:
      tf-api-token: ${{ secrets.TF_API_TOKEN }}
```
The saved plan is uploaded as a one-day artifact and may contain sensitive values. Cloud credentials are not handled here; use a remote backend with its own authentication or a Terraform Cloud token.

## Notes
- The caller must grant at least the `permissions` that the called workflow's job needs (the examples above already do).
- This repo must be public, or private with *Settings > Actions > General > Access* allowing other repos to use it.
