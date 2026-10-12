# Terraform workflows

GitHub only loads reusable workflows from `.github/workflows/` (no subfolders), so the
workflow files live there with a `terraform-` prefix. This folder holds the documentation
and ready-to-copy caller examples.

## Building blocks

Each workflow does one job, with its own permissions, so callers compose only what they need.

| Workflow | Purpose | Credentials | Typical trigger |
| --- | --- | --- | --- |
| `terraform-validate.yml` | `fmt`, `validate`, TFLint | none | `pull_request` (forks included) |
| `terraform-security.yml` | Trivy misconfiguration scan, SARIF to code scanning | none | `pull_request`, `push` |
| `terraform-plan.yml` | Plan with OIDC, sticky PR comment | OIDC (read-only role) | `pull_request` |
| `terraform-apply.yml` | Plan, approval gate, apply the saved plan | OIDC (plan role + apply role) | `push` to `main` |
| `terraform-drift.yml` | Scheduled plan, opens or closes a drift issue | OIDC (read-only role) | `schedule` |
| `terraform-destroy.yml` | Typed confirmation, approval gate, destroy | OIDC | `workflow_dispatch` |

## Recommended flow

```
pull request                          push to main
  validate  (no credentials)            validate + security
  security  (no credentials)               |
  plan      (read-only role)            apply workflow
                                          plan job   (read-only role)
                                             |  plan artifact + sha256
                                          environment approval
                                             |
                                          apply job  (write role, applies that exact plan)
```

## Security model

- **No long-lived cloud keys.** Authentication uses GitHub OIDC for AWS, GCP and Azure.
- **Least privilege per job.** Workflow-level `permissions: {}`; each job requests only what it needs. `id-token: write` exists only in jobs that talk to the cloud.
- **Split identities.** `terraform-apply.yml` and `terraform-destroy.yml` run the plan job with `plan-*` inputs (a read-only role) and the apply job with the write role.
- **Apply what was reviewed.** The plan is saved, checksummed (`sha256`) and the apply job verifies it before applying. Terraform itself rejects a stale plan.
- **Approval gate.** Pass `environment` and configure required reviewers on that environment.
- **Ref allow-list.** Apply and destroy refuse to run unless `github.ref` equals `allowed-ref` (default `refs/heads/main`).
- **No overlapping runs.** Apply and destroy share a concurrency group per working directory and environment, with `cancel-in-progress: false`.
- **Credentials not persisted.** `actions/checkout` runs with `persist-credentials: false`.
- **Pinned scanner.** The Trivy action is pinned to a commit SHA.

### Things to know

- The plan artifact (one-day retention) is the real plan file and can contain sensitive values. Restrict who can read workflow artifacts.
- Fork pull requests receive no OIDC token, so plan cannot run for them. Only `validate` and `security` are credential-free.
- Pin `terraform-version` to an exact version in callers for reproducible runs.

## One-time cloud setup (OIDC trust)

Restrict the trust policy to your repository and, for the write role, to the GitHub environment.

AWS (IAM role trust policy, apply role):

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com" },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": {
        "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
        "token.actions.githubusercontent.com:sub": "repo:<OWNER>/<REPO>:environment:production"
      }
    }
  }]
}
```

For the read-only plan role, allow `repo:<OWNER>/<REPO>:ref:refs/heads/main` and
`repo:<OWNER>/<REPO>:pull_request` and attach only read permissions.

GCP: bind the workload identity pool principal for the repository to the service account
(`roles/iam.workloadIdentityUser`) and add an attribute condition on `assertion.repository`.

Azure: add federated credentials on the app registration for the `environment:<name>`,
`ref:refs/heads/main` and `pull_request` subjects.

## Examples

| File | Use |
| --- | --- |
| [`examples/terraform-pr.yml`](examples/terraform-pr.yml) | Validate, scan and plan on pull requests |
| [`examples/terraform-deploy.yml`](examples/terraform-deploy.yml) | Apply on push to `main`, behind an environment |
| [`examples/terraform-drift.yml`](examples/terraform-drift.yml) | Nightly drift detection |
| [`examples/terraform-destroy.yml`](examples/terraform-destroy.yml) | Manual, confirmed destroy |

Copy a file into `.github/workflows/` of the calling repo and replace `OWNER` and the cloud IDs.
