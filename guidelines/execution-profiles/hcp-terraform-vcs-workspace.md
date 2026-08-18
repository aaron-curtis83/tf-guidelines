---
title: HCP Terraform VCS Workspace Profile
description: Use HCP Terraform workspace runs when version control drives Terraform execution
---

## Profile Boundary

[Execution profile] Select this profile when an HCP Terraform workspace is
connected to version control and owns Terraform runs for its configuration and
state boundary. The workspace run model, policy checks, approvals, and state
handling are HCP Terraform responsibilities.

## VCS Run Lifecycle

[Execution profile] A pull request can produce a speculative plan for review.
After merge, the VCS workspace creates a fresh normal run from the merged
configuration. It does not apply the speculative pull-request plan.

Review the normal run according to the workspace's configured approval and
policy behavior before an apply proceeds. Retain the run and policy outcomes as
the delivery evidence for the merged configuration.

## State and Ownership

Assign each independently deployable root to the intended workspace and state
boundary. Keep workspace ownership, variable management, policy sets, and
approval responsibilities explicit before handing the configuration to an
operator.

The organization has not yet approved a universal HCP Terraform edition,
workspace naming convention, retention period, or approval policy. Record the
chosen settings as local delivery decisions until the relevant owner publishes
an organization baseline.

## Platform Controls

Use the [HCP Terraform overlay](../platform-overlays/hcp-terraform.md) for
workspace, run, policy, and approval behavior. Do not import saved-plan
artifact selection, GitHub environments, or GitHub reusable-workflow mechanics
into this profile.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable evidence expectations and
[handover evidence packs](../core/handover-evidence-pack.md) for operator
handover records.

## Workspace Mapping

[Execution profile] Map one independently deployable root to its workspace.

Record repository, configuration directory, tracked branch, and owner.

Do not rely on workspace name alone.

| Root | Workspace | Directory | Branch | State owner |
| --- | --- | --- | --- | --- |
| `roots/networking` | `networking-prod` | `roots/networking` | `main` | Networking owner |
| `roots/payments` | `payments-prod` | `roots/payments` | `main` | Payments owner |

The table is original example data, not a required naming convention.

[Gap] Edition, naming, project assignment, and ownership baselines require an
approved organization decision.

## VCS Lifecycle

[Execution profile] A VCS workspace can create a speculative plan for a PR.

[Execution profile] A tracked-branch merge creates a fresh normal run.

[Execution profile] The normal run represents the merged commit.

[Execution profile] It is not application of the speculative PR plan.

Use HashiCorp [VCS-driven runs](https://developer.hashicorp.com/terraform/cloud-docs/workspaces/run/ui)
documentation for run behavior.

```text
pull request commit
	-> speculative run for review
	-> merge to tracked branch
	-> normal run for merged commit
	-> policy evaluation and approval behavior
	-> apply or no apply
```

> [!WARNING]
> Do not apply an HCP Terraform speculative pull-request plan after merge.
> Locate the fresh normal run for the merged commit and review that run.

## Approval and Apply

A speculative run informs pull-request review.

A normal run is the post-merge delivery candidate.

[Execution profile] Authorized users can confirm a normal run when auto-apply
is not enabled.

[Execution profile] Auto-apply can proceed according to configured workspace
behavior after normal-run and policy processing.

Record manual confirmation or auto-apply for each workspace.

Do not infer manual approval from workspace existence.

[Assumption] Workspace permissions and auto-apply are reviewed by the owner.

Local evidence does not verify live settings.

## Policy Evidence

[Execution profile] Policy evidence belongs to the normal run for the merge.

Policy sets can be global, project-scoped, or workspace-scoped.

Their enforcement result can stop a run from applying.

Use HashiCorp [policy enforcement](https://developer.hashicorp.com/terraform/cloud-docs/policy-enforcement)
documentation for supported behavior.

Record policy-set identifier, scope, result, enforcement level, and exceptions.

Do not infer a required policy framework from feature support.

[Gap] Policy assignment, exceptions, enforcement, and retention are
organization-specific decisions.

## Variables and Sensitivity

[Execution profile] Workspace variables provide run inputs for this state.

Record variable name, classification, source owner, and rotation contact.

Do not copy sensitive values into handover evidence.

Classify values as configuration, sensitive input, or inherited setting.

> [!CAUTION]
> Sensitive workspace values and run output can expose operational data.
> Limit view and edit access to authorized operators.

[Practice recommendation] Include variable metadata, not secret values, in
handover evidence.

## Non-Combinable Controls

Do not configure a CI runner to apply state owned by this workspace.

Do not download a speculative run as a runner-managed artifact.

Do not create a local post-merge plan to replace the normal run.

CI can validate and test without becoming a second apply authority.

Record workspace identifier in CI evidence to expose this boundary.

## Normal-Run Procedure

1. Confirm root-to-workspace mapping.
2. Confirm directory and tracked branch.
3. Locate normal run for merged revision.
4. Confirm run revision matches delivery change.
5. Review that normal run's plan.
6. Review policy outcome.
7. Confirm manual or auto-apply behavior.
8. Apply only through authorized workspace path.
9. Record status, run locator, and state owner.

Stop for a different root, directory, workspace, or commit.

Do not fall back to speculative run when normal run is absent.

## Original Evidence Record

```text
root: roots/networking
workspace: networking-prod
repository: platform-infrastructure
vcs_directory: roots/networking
tracked_branch: main
merged_revision: 8c4f6a1
speculative_review_run: run-4pL8
normal_run: run-7mQ2
policy_set: baseline-networking
policy_result: passed
policy_enforcement: mandatory
apply_mode: manual
approval_record: workspace audit event
apply_status: applied
```

The speculative run is review context, not the applied plan.

Do not add tokens, credentials, or variable values.

## Failure Modes

| Failure | Consequence | Required response |
| --- | --- | --- |
| PR plan applied after merge | No merged-run provenance | Locate normal run |
| Wrong VCS directory | Wrong root can run | Correct mapping |
| Wrong normal revision | Review differs from delivery | Stop and find commit |
| Runner also applies state | Conflicting authorities | Disable one path |
| Policy ignored | Governance bypassed | Resolve or record exception |
| Auto-apply assumed | Authority unclear | Record configuration |
| Secret in handover | Confidentiality failure | Remove values |

## Verification and Handover

Verify root, workspace, repository, directory, branch, normal run, policy
result, apply mode, permission owner, and state recovery contact.

Record normal run, merged revision, policy outcome, approval evidence, and
variable metadata in the [handover evidence pack](../core/handover-evidence-pack.md).

Use [known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
for unresolved retention, recovery, edition, and policy decisions.