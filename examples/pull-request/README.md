# Pull request template

The template lives at [`.github/PULL_REQUEST_TEMPLATE.md`](../../.github/PULL_REQUEST_TEMPLATE.md).

## What it contains

| Section | Purpose |
| --- | --- |
| Summary | What the PR does and why; link issues with `Closes #123` |
| Changes | Short list of the key changes |
| Type of change | Feature, Bug fix, Breaking change, Refactor, Documentation, Chore |
| Testing | How it was verified; link a workflow run when there is one |
| Checklist | Title format, checks, docs, secrets, breaking changes |
| Notes | Breaking changes, migration steps, anything reviewers should know |

## How it is used

### In this repository

Nothing to do. GitHub pre-fills the PR description with the template whenever a PR is opened from the web UI against the default branch.

### With the GitHub CLI

`gh pr create` does not fill the description from the template when `--body` is given. Use one of:

```bash
# Interactive: opens the editor with the template as starting text
gh pr create --template .github/PULL_REQUEST_TEMPLATE.md

# Non-interactive: write the filled body to a file first
gh pr create --title "feat(ci-node): add cache-dependency-path input" --body-file pr-body.md
```

### In other repositories

Pick one:

1. **Copy the file** to `.github/PULL_REQUEST_TEMPLATE.md` in each repository.
2. **Organization default**: put the file in the root of a repository named `.github` in the organization (or personal account). It applies to every repository that has no template of its own.

### Multiple templates

For different kinds of PRs, add files under `.github/PULL_REQUEST_TEMPLATE/` and select one with a query parameter:

```
https://github.com/OWNER/REPO/compare/main...my-branch?template=docs.md
```

## Filling it in

1. Write the title as a Conventional Commit: `type(scope): description`, with `!` for breaking changes (`feat(ci-node)!: rename scripts input`). The [`automation-pr-title-lint.yml`](../../.github/workflows/automation-pr-title-lint.yml) workflow can enforce this.
2. Tick one type of change, and tick checklist items only when they are true.
3. Delete the Notes section when there is nothing to add.
4. Prefer evidence in Testing: a link to a passing run is better than a statement.

## Examples

| File | Scenario |
| --- | --- |
| [`feature.md`](feature.md) | New input on a reusable workflow |
| [`breaking-change.md`](breaking-change.md) | Renamed input, with migration steps |
| [`bug-fix.md`](bug-fix.md) | Fix with regression evidence |
