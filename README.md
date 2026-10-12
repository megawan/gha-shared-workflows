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

## Usage examples

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

## Notes
- The caller must grant at least the `permissions` that the called workflow's job needs (the examples above already do).
- This repo must be public, or private with *Settings > Actions > General > Access* allowing other repos to use it.
