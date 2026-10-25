# Contributing and new repository rules

Two audiences: people who **add or change a workflow in this repository** (Part 1), and people who **set up a new repository that uses these workflows** (Part 2).

## Part 1: Adding a workflow to this repository

### Naming and layout

- Workflow files go in `.github/workflows/` (GitHub does not load subfolders) and start with a group prefix: `automation-`, `ci-`, `security-`, `site-` or `terraform-`. Use kebab-case: `<prefix>-<what>.yml`.
- A new group needs a new prefix, a section in [`README.md`](README.md), and an `examples/<group>/` folder.
- Every reusable workflow ships with a ready-to-copy caller in `examples/<group>/`, using `OWNER/gha-shared-workflows/...@v1`.
- Add the workflow to the table in [`README.md`](README.md): purpose and caller trigger.

### Workflow file

- Trigger only on `workflow_call`. Start the file with a comment saying what it does and which caller triggers are typical.
- Set `permissions: {}` at the top level, then grant the minimum per job (for example `contents: read`). Never grant `write` unless the job needs it.
- Every `input` has a `description`, a `type` and a sensible `default`. Add inputs rather than hardcoding paths, versions or tool flags.
- Pass inputs and `github.*` context to scripts through `env:`, never by interpolating `${{ }}` into a `run:` block (script injection).
- Use `actions/checkout` with `persist-credentials: false` unless the job pushes.
- Secrets are declared under `on.workflow_call.secrets`. Prefer OIDC over long-lived cloud keys.
- Third-party actions: pin to a major tag (`@v4`) like the existing workflows; pin to a full commit SHA with a `# vX.Y.Z` comment for actions from smaller or less trusted publishers. Dependabot keeps both up to date.
- Do not use `pull_request_target` to check out or run code from a PR head.

### Before opening a PR

- `lint.yml` (actionlint) must pass. It runs on every change under `.github/workflows/`.
- Try the workflow from a caller before merging, and link the run in the PR's Testing section.
- Use the [PR template](.github/PULL_REQUEST_TEMPLATE.md). The title follows Conventional Commits and is checked by `pr-title.yml`: `feat(site): add smoke workflow`. Use the group as scope.
- One workflow (or one fix) per PR.

### Versioning

- Callers pin `@v1`. Backward-compatible changes (new optional input, bug fix) go into `v1`.
- Breaking changes (removed or renamed input, new required input or secret, higher permissions, changed behavior) need a new major tag, a `feat!:`/`BREAKING CHANGE` title, and migration notes in the PR.

## Part 2: Setting up a new repository that uses these workflows

### Required

1. **Pin by tag**: `uses: OWNER/gha-shared-workflows/.github/workflows/<file>@v1`. Never `@main`.
2. **Grant permissions in the caller**: each calling job sets `permissions:` to at least what the called workflow needs. Copy from `examples/`. Add `permissions: {}` at workflow level.
3. **Default branch**: `main`. Callers use `pull_request` plus `push` to `main`.
4. **PR titles**: add [`automation-pr-title-lint`](examples/automation/pr-title-lint.yml) and use Conventional Commits.
5. **Workflow lint**: add [`automation-actionlint`](examples/automation/actionlint.yml).
6. **Secret scan**: add [`security-secret-scan`](examples/security/secret-scan.yml). Never commit secrets; use GitHub Secrets or OIDC.
7. **Dependabot**: add `.github/dependabot.yml` with the `github-actions` ecosystem (plus the project's own ecosystem, such as `npm`, `pip`, `gomod` or `docker`).
8. **PR template**: copy [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md), or rely on an organization-level default.
9. **CI for the language in use**: one of [`ci-node`, `ci-python`, `ci-go`, `ci-docker-build-push`](examples/ci).

### By project type

| Type | Add |
| --- | --- |
| Application or library | CI workflow for the language, [`security-codeql`](examples/security/codeql.yml), [`security-dependency-review`](examples/security/dependency-review.yml) |
| Static site | [`site-*`](examples/site) workflows (html-validate, link-check, smoke, lighthouse) |
| Infrastructure | [`terraform-*`](examples/terraform) workflows, following [its README](examples/terraform/README.md): OIDC, protected `apply`/`destroy` environments, no static cloud keys |
| Container image | [`ci-docker-build-push`](examples/ci/docker.yml); push only from `main` or tags |

### Repository settings

- Protect `main`: require PRs, require the status checks above, block force pushes and deletions.
- Squash merge with the PR title as the commit message, so history stays Conventional Commits.
- Enable "Automatically delete head branches".
- Enable Dependabot alerts and secret scanning (with push protection) if the plan allows it.
- Set the default `GITHUB_TOKEN` permission to read-only (*Settings > Actions > General*).
- Create `terraform` environments with required reviewers before using apply or destroy.
- If the repo is private, make sure this repository's *Settings > Actions > General > Access* allows it to call these workflows.
- Optional: [`automation-labeler`](examples/automation/labeler.yml) and [`automation-sync-labels`](examples/automation/sync-labels.yml) for labels; [`automation-stale`](examples/automation/stale.yml) for inactive issues; [`automation-auto-merge-dependabot`](examples/automation/auto-merge-dependabot.yml) for low-risk updates.

### Checklist

- [ ] Every `uses:` points to `@v1`, not `@main`
- [ ] Callers set minimum `permissions`
- [ ] PR title lint, actionlint and secret scan are enabled
- [ ] `dependabot.yml` covers `github-actions` and the project's ecosystem
- [ ] PR template is in place
- [ ] Language CI (and site or terraform checks when relevant) is enabled
- [ ] `main` is protected and the checks are required
- [ ] No secrets in the repo; cloud access uses OIDC
