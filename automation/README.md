# PR and repo automation workflows

Workflow files live in `.github/workflows/` (GitHub requires a flat layout) and share the `automation-` prefix.

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`automation-pr-title-lint.yml`](../.github/workflows/automation-pr-title-lint.yml) | Validate PR titles (Conventional Commits) | `pull_request_target` |
| [`automation-labeler.yml`](../.github/workflows/automation-labeler.yml) | Label PRs automatically by changed paths | `pull_request_target` |
| [`automation-stale.yml`](../.github/workflows/automation-stale.yml) | Mark and close inactive issues and PRs | `schedule`, `workflow_dispatch` |
| [`automation-auto-merge-dependabot.yml`](../.github/workflows/automation-auto-merge-dependabot.yml) | Approve and auto-merge Dependabot PRs | `pull_request` |
| [`automation-sync-labels.yml`](../.github/workflows/automation-sync-labels.yml) | Sync repo labels with a definition file | `push`, `workflow_dispatch` |
| [`automation-actionlint.yml`](../.github/workflows/automation-actionlint.yml) | Lint workflow files with actionlint (checksum-verified binary, shellcheck included) | `pull_request`, `push` |

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
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-pr-title-lint.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-labeler.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-stale.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-auto-merge-dependabot.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-sync-labels.yml@v1
```

### Actionlint
Lints every file in `.github/workflows/` of the caller repo.
```yaml
name: Lint
on:
  pull_request:
    paths: [".github/workflows/**"]
jobs:
  actionlint:
    permissions:
      contents: read
    uses: OWNER/gha-shared-workflows/.github/workflows/automation-actionlint.yml@v1
```

This repository lints itself with [`lint.yml`](../.github/workflows/lint.yml), which calls the same workflow.
