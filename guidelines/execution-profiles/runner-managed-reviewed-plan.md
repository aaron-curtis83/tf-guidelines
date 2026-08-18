---
title: Runner-Managed Reviewed Plan Profile
description: Use an exact saved Terraform plan when a CI runner owns planning and applying
---

## Profile Boundary

[Execution profile] Select this profile when a CI runner creates a binary
Terraform plan for a configuration and later applies that same plan. The
runner, not a Terraform workspace service, owns artifact retention, selection,
identity use, and apply orchestration.

## Preserve Plan Provenance

Associate the saved plan with the configuration revision, the state boundary,
and the successful validation run that created it. Before apply, retrieve the
intended artifact and verify that it contains the expected binary plan for the
same configuration and state coordinates.

[Execution profile] Applying a saved plan proves only that the runner selected
and applied that artifact. It does not prove that a reviewer approved it, that
branch protection passed, or that an independent deployment approval occurred.
Those are separate governance controls.

## Apply and Destructive Changes

Apply the exact saved binary plan rather than replacing it with a fresh plan
in the target branch. Keep lifecycle operations that affect the same state
boundary serialized. Treat destruction as a distinct, manually initiated
operation with its own reviewed plan and authorization controls.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
to define review evidence and [state and boundaries](../core/state-and-boundaries.md)
to identify the serialized state boundary.

## Saved Plan Semantics

[Terraform universal] `terraform plan -out=PLANFILE` writes a binary plan.

[Terraform universal] `terraform apply PLANFILE` applies that stored plan.

It does not create a fresh plan for the current branch state.

Use HashiCorp [plan](https://developer.hashicorp.com/terraform/cli/commands/plan)
and [apply](https://developer.hashicorp.com/terraform/cli/commands/apply)
documentation for command semantics.

[Execution profile] The selected binary plan is the sole apply input.

Do not re-plan after merge as a convenience step.

If replacement is necessary, restart review and evidence collection.

## Provenance Invariants

Record configuration revision.

Record root path and state identifier.

Record validation run identifier.

Record artifact name and plan filename.

Record binary plan checksum.

Record review, approval, and policy locators.

Select artifacts by these fields, not a latest-run query.

[Execution profile] Applying a saved plan proves artifact selection only.

It does not independently prove review, signing, approval, or policy.

## Original Manifest

```yaml
plan_evidence:
  root_path: roots/payments
  state_identifier: payments-prod
  pull_request: 284
  revision: 8c4f6a1
  validation_run: 912
  artifact_name: payments-prod-plan
  plan_file: terraform.tfplan
  checksum_sha256: 6f5aa9d0...
```

This is original pseudocode, not an artifact-service contract.

[Gold-standard example] Current GitHub research matches a successful PR run by
`head_sha`, retrieves its named artifact, verifies the plan, and applies it
without re-planning after merge.

That observation does not prove artifact signing or reviewer approval.

## Plan Procedure

1. Acquire same-state coordination.
2. Initialize the intended backend.
3. Run validation and assurance checks.
4. Create a binary plan for the intended root.
5. Produce permitted review output.
6. Calculate the binary-plan checksum.
7. Store artifact and manifest together.
8. Record revision and validation run.
9. Submit evidence to configured review.

```bash
terraform init
terraform validate
terraform plan -out=terraform.tfplan
Get-FileHash terraform.tfplan -Algorithm SHA256
```

The PowerShell hash is local verification, not artifact attestation.

## Apply Procedure

1. Locate artifact by revision and validation run.
2. Verify root path and state identifier.
3. Download the named artifact.
4. Confirm the plan file exists.
5. Verify checksum against manifest.
6. Confirm review, policy, and approval evidence.
7. Acquire same-state serialization.
8. Initialize the matching backend.
9. Apply the exact plan file.
10. Record outcome and timestamp.

```bash
Get-FileHash terraform.tfplan -Algorithm SHA256
terraform apply terraform.tfplan
```

> [!CAUTION]
> Do not replace the selected runner-managed plan with a fresh post-merge
> plan. The replacement breaks the reviewed artifact provenance chain.

## Artifact Sensitivity

Plan files can contain sensitive values in cleartext.

Do not commit plan files to source control.

Do not publish plans in comments or build logs.

Limit creation and download permissions.

Treat rendered summaries as sensitive operational evidence.

[Gap] Retention, encryption, attestation, and recovery controls are
organization-specific decisions.

## Same-State Serialization

[Execution profile] Serialize plan, apply, drift, and destroy for one state.

[Terraform universal] Backend locking can reduce concurrent state writes.

Locking does not select the artifact or authorize an apply.

Use a platform coordination mechanism scoped to the state identifier.

Record the coordination key and cancellation owner.

## Stale or Missing Artifact

| Condition | Do not do | Required response |
| --- | --- | --- |
| State changed | Force old plan | Investigate and create new reviewed plan |
| Revision differs | Reuse by branch | Re-plan and repeat review |
| Artifact missing | Create unrecorded main plan | Block and restart cycle |
| Checksum differs | Apply anyway | Reject and investigate |
| State differs | Retarget plan | Stop and correct selection |
| Active operation | Run in parallel | Wait or resolve coordination |

## Separate Destroy Flow

[Execution profile] Destroy is manually initiated and separately authorized.

Create a new destroy plan for the intended root and state.

Review that destroy plan separately.

Apply only that saved destroy plan.

```bash
terraform plan -destroy -out=terraform.destroy.tfplan
terraform apply terraform.destroy.tfplan
```

Do not reuse a normal plan for destruction.

Do not start destroy during another same-state operation.

## Adapter Boundaries

| Concern | This profile | Overlay |
| --- | --- | --- |
| Plan semantics | Exact saved plan | Command implementation |
| Selection | Provenance facts | Artifact API |
| Serialization | Same-state requirement | Concurrency configuration |
| Identity | Record identities | OIDC or service connection |
| Approval | Record evidence | Platform checks |
| Destroy | Separate plan | Manual trigger details |

Use [GitHub Actions](../platform-overlays/github-actions.md) or
[Azure DevOps](../platform-overlays/azure-devops.md) for adapter mechanics.

## Verification and Handover

Verify root, state, revision, validation run, artifact, checksum, coordination,
approval evidence, policy evidence, and exact apply command.

Record manifest, apply result, and recovery owner in the
[handover evidence pack](../core/handover-evidence-pack.md).

Do not include credentials or plan contents in handover records.