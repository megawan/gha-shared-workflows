**Title:** `feat(ci-node): add working-directory input`

## Summary

Allows `ci-node.yml` to run in a subdirectory, so monorepos can call it once per package.

Closes #42

## Changes

- Add `working-directory` input (default `.`)
- Apply it to the install and script steps
- Add a monorepo example to `examples/ci/node.yml`

## Type of change

- [x] Feature
- [ ] Bug fix
- [ ] Breaking change
- [ ] Refactor
- [ ] Documentation
- [ ] Chore

## Testing

- `actionlint` passes
- Run against `apps/web` in a monorepo: https://github.com/OWNER/REPO/actions/runs/123456789
- Default behavior unchanged for repos that do not set the input

## Checklist

- [x] Title follows Conventional Commits
- [x] Tests or checks pass
- [x] Documentation and examples are updated
- [x] No secrets or sensitive data are included
- [x] Breaking changes are described below

## Notes

No breaking changes. Safe for the current major tag.
