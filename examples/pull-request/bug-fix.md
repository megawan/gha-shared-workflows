**Title:** `fix(automation-stale): do not close issues with exempt labels`

## Summary

Issues labeled `pinned` were closed by the stale workflow because `exempt-labels` was passed only for PRs.

Closes #57

## Changes

- Pass `exempt-labels` to both `exempt-issue-labels` and `exempt-pr-labels`

## Type of change

- [ ] Feature
- [x] Bug fix
- [ ] Breaking change
- [ ] Refactor
- [ ] Documentation
- [ ] Chore

## Testing

- Dry run (`dry-run: true`) on a repo with a `pinned` issue: the issue is skipped (https://github.com/OWNER/REPO/actions/runs/123456789)
- Before the fix the same run listed it as stale

## Checklist

- [x] Title follows Conventional Commits
- [x] Tests or checks pass
- [x] Documentation and examples are updated
- [x] No secrets or sensitive data are included
- [x] Breaking changes are described below

## Notes

Behavior change only for issues carrying an exempt label, which is the documented intent.
