# CI workflows

Workflow files live in `.github/workflows/` (GitHub requires a flat layout) and share the `ci-` prefix.

| Workflow | Purpose | Caller trigger |
| --- | --- | --- |
| [`ci-node.yml`](../.github/workflows/ci-node.yml) | Install, lint, test and build with npm, pnpm or yarn | `push`, `pull_request` |
| [`ci-python.yml`](../.github/workflows/ci-python.yml) | Install, optional lint, and test with pytest | `push`, `pull_request` |
| [`ci-go.yml`](../.github/workflows/ci-go.yml) | `go vet`, golangci-lint, `go test -race`, `go build` | `push`, `pull_request` |
| [`ci-docker-build-push.yml`](../.github/workflows/ci-docker-build-push.yml) | Build and push (multi-arch) images with metadata tags and cache | `push`, `pull_request`, tags |

## Usage examples

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
    uses: OWNER/gha-shared-workflows/.github/workflows/ci-node.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/ci-python.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/ci-go.yml@v1
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
    uses: OWNER/gha-shared-workflows/.github/workflows/ci-docker-build-push.yml@v1
    with:
      platforms: linux/amd64,linux/arm64
```
For other registries, set `registry` and pass the `registry-username` and `registry-password` secrets.


