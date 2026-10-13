**Title:** `feat(terraform-apply)!: rename environment input to deploy-environment`

## Summary

`environment` is ambiguous next to `cloud-provider`. Renaming it to `deploy-environment` makes the intent clear and aligns with `terraform-destroy.yml`.

## Changes

- Rename input `environment` to `deploy-environment` in `terraform-apply.yml`
- Update `examples/terraform/deploy.yml`
- Update the Terraform README

## Type of change

- [ ] Feature
- [ ] Bug fix
- [x] Breaking change
- [ ] Refactor
- [ ] Documentation
- [ ] Chore

## Testing

- `actionlint` passes
- Apply run in a sandbox repo with the new input name: https://github.com/OWNER/REPO/actions/runs/123456789

## Checklist

- [x] Title follows Conventional Commits
- [x] Tests or checks pass
- [x] Documentation and examples are updated
- [x] No secrets or sensitive data are included
- [x] Breaking changes are described below

## Notes

**Breaking change.** Callers must rename the input. Requires a new major tag (`v2`).

```yaml
# before
with:
  environment: production

# after
with:
  deploy-environment: production
```
