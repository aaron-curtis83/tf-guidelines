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

## Run Stages and Evidence

[HCP Terraform] A run progresses through queued, plan, optional cost and
policy checks, apply, and completion stages. Optional run-task stages can also
occur before or after planning and applying. The stages that actually occur
depend on the workspace and organization configuration. See HashiCorp's
[run states and stages documentation](https://developer.hashicorp.com/terraform/cloud-docs/run/states).

Record the normal run's observed stages, not a claimed universal workflow.
For example, a workspace without enabled policy sets does not produce policy
evidence, while a workspace with a policy check can pause or fail according to
its enforcement level.

| Stage or outcome | Operator decision | Handover evidence |
| --- | --- | --- |
| Pending or fetching | Confirm the normal run targets the intended workspace and merged revision | Workspace and run locator |
| Plan complete | Review proposed actions for the normal run | Plan view and revision |
| Policy check | Record every applied policy-set result and any permitted override | Policy result, enforcement, override locator |
| Needs confirmation | Confirm or discard only through an authorized workspace path | Manual confirmation or discard record |
| Auto-apply path | Record that the workspace applied according to its configured behavior | Workspace setting reference and applied run |
| Applied or planned and finished | Record the terminal result and verification reference | Final run state and timestamp |
| Plan or apply errored | Stop automatic progression and route to the named owner | Error locator, state owner, next decision |

[HCP Terraform] Policy sets can apply globally or at project or workspace
scope. HCP Terraform evaluates the plan against policy sets that apply to the
workspace, and enforcement results can stop or pause a run. See HashiCorp's
[policy enforcement overview](https://developer.hashicorp.com/terraform/cloud-docs/policy-enforcement).

[Gap] The existence of a policy feature does not establish that this
organization assigns a particular policy set, permits overrides, requires
manual confirmation, or enables auto-apply for any workspace.

## Failure Handling

Use the terminal HCP run state and workspace evidence to decide the next
action. Do not create a runner-managed plan to bypass a blocked workspace run.

| Condition | Immediate response | Do not do |
| --- | --- | --- |
| Speculative run is stale after the target branch changes | Review the appropriate normal run after merge or update PR review context | Apply the older speculative result |
| VCS fetch or plan fails | Correct the configuration, variables, or VCS mapping and let a new run represent the corrected input | Reclassify the failed run as approved |
| Policy check fails | Follow the configured enforcement and authorized override or discard path | Treat a policy failure as advisory without evidence |
| Plan awaits confirmation | Identify the authorized workspace path and record the decision | Infer approval from PR status |
| Apply errors or is canceled | Stop follow-on runs, assess state with the state owner, and record the next recovery decision | Start a second apply authority or edit state directly |
| A normal run is absent for a merged revision | Inspect workspace VCS mapping and trigger evidence, then use the workspace's supported path | Substitute the PR speculative run |

## Original Run Handover Record

Use this original record to preserve the distinction between PR review and the
normal run that could apply the merged revision.

```text
root: roots/networking
workspace: networking-production
workspace_locator: <approved-workspace-reference>
repository: platform-infrastructure
working_directory: roots/networking
tracked_branch: main
merged_revision: 8c4f6a1
speculative_review_run: <optional-pr-run-locator>
normal_run: <required-normal-run-locator>
normal_run_terminal_state: applied | planned-and-finished | errored | discarded
policy_results: <applied-policy-result-locators-or-none-observed>
apply_decision: manual-confirmation | auto-apply | discarded
state_owner: <role-or-escalation-reference>
recovery_contact: <role-or-escalation-reference>
```

Do not include variable values, tokens, state data, or plan output. A listed
speculative run supplies review context only and is not evidence that HCP
Terraform applied that PR plan.

## Verification and Handover

Verify root, workspace, repository, directory, branch, normal run, policy
result, apply mode, permission owner, and state recovery contact.

Record normal run, merged revision, policy outcome, approval evidence, and
variable metadata in the [handover evidence pack](../core/handover-evidence-pack.md).

Use [known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
for unresolved retention, recovery, edition, and policy decisions.