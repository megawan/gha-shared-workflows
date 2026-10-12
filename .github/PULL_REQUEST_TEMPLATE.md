<!--
  PR title: use Conventional Commits, e.g. `feat(terraform): add plan-args input`
  Types: feat, fix, docs, refactor, perf, test, ci, chore, revert
  Mark breaking changes with `!`, e.g. `feat(ci-node)!: rename scripts input`
  Delete any section that does not apply, but keep the headings you fill in.
-->

## Summary

<!-- What does this change do and why? One short paragraph. Link issues with "Closes #123". -->

## Type of change

- [ ] New reusable workflow
- [ ] New input, output or secret (backward compatible)
- [ ] Bug fix
- [ ] Breaking change (input, output, secret, permission or behavior change)
- [ ] Dependency or action version bump
- [ ] Documentation or examples only
- [ ] Repository tooling (lint, release, templates)

## Affected workflows

<!-- Tick every file this PR touches or whose behavior it changes. -->

**Automation**
- [ ] `automation-pr-title-lint.yml`
- [ ] `automation-labeler.yml`
- [ ] `automation-stale.yml`
- [ ] `automation-auto-merge-dependabot.yml`
- [ ] `automation-sync-labels.yml`
- [ ] `automation-actionlint.yml`

**CI**
- [ ] `ci-node.yml`
- [ ] `ci-python.yml`
- [ ] `ci-go.yml`
- [ ] `ci-docker-build-push.yml`

**Terraform**
- [ ] `terraform-validate.yml`
- [ ] `terraform-security.yml`
- [ ] `terraform-plan.yml`
- [ ] `terraform-apply.yml`
- [ ] `terraform-drift.yml`
- [ ] `terraform-destroy.yml`

**Other**
- [ ] `examples/`
- [ ] `README.md`
- [ ] Repository tooling (`lint.yml`, templates)

## Interface changes

<!--
  The inputs, outputs, secrets and permissions are the public API of a reusable workflow.
  List every change. Write "None" if the interface is untouched.
-->

| Workflow | Kind | Name | Change | Default | Breaking? |
| --- | --- | --- | --- | --- | --- |
| `ci-node.yml` | input | `example-input` | added | `""` | No |

## Caller impact

<!--
  What must users of this workflow change, if anything?
  For breaking changes, give the exact before/after snippet and the migration steps.
-->

- [ ] No action needed by callers
- [ ] Callers must update their `with:` / `secrets:` / `permissions:`
- [ ] Callers must change repository or environment settings

```yaml
# before

# after
```

## Security review

<!-- Reusable workflows run with the caller's permissions and secrets. Answer honestly. -->

- [ ] Job permissions are the minimum needed and `permissions: {}` is set at workflow level
- [ ] No new secret, token or credential is required, or it is declared explicitly under `secrets:`
- [ ] No `${{ ... }}` expression is interpolated directly into `run:` scripts; values go through `env:`
- [ ] Third-party actions are pinned (major tag from a trusted publisher, or full commit SHA otherwise)
- [ ] Cloud access uses OIDC, not long-lived keys
- [ ] No sensitive value is written to logs, summaries, comments or artifacts
- [ ] Behavior on pull requests from forks has been considered

**Notes:** <!-- Anything a reviewer should look at closely, or "None". -->

## Testing

<!-- Show how you verified this. Linking a successful run is the strongest evidence. -->

- [ ] `actionlint` passes locally or in the `Lint` check
- [ ] Exercised from a real caller workflow

| Scenario | Caller or branch | Result | Run |
| --- | --- | --- | --- |
| Default inputs | | | |
| Changed or new inputs | | | |
| Failure path | | | |

**Not tested, and why:** <!-- Or "Nothing". -->

## Documentation

- [ ] Workflow `name`, header comment and input `description` fields are accurate
- [ ] `README.md` workflow tables updated
- [ ] A caller example exists or was updated in `examples/`
- [ ] `examples/terraform/README.md` updated (Terraform changes only)

## Release

- [ ] Safe to include in the current major tag (`v1`)
- [ ] Requires a new major tag (`v2`), because of the breaking change above
- [ ] Release notes entry drafted below

<!-- Release note (one line, user-facing): -->

## Reviewer notes

<!-- Where to start reading, trade-offs you considered, follow-ups you intentionally left out. -->
