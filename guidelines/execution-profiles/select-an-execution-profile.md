---
title: Select an Execution Profile
description: Choose the Terraform execution authority before following platform-specific delivery guidance
---

## Choose One Apply Authority

Select one profile for each independently deployable configuration before
following a platform overlay. The profile determines how a reviewed change,
apply action, and delivery evidence relate to one another.

| When the configuration is delivered by | Select this profile |
| --- | --- |
| A CI runner that saves and later applies a reviewed binary plan | [Runner-managed reviewed plan](runner-managed-reviewed-plan.md) |
| An HCP Terraform workspace connected to version control | [HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md) |
| A CI runner that applies a saved binary plan | [Runner-managed reviewed plan](runner-managed-reviewed-plan.md) |
| An HCP Terraform workspace that creates a new run from the merged configuration | [HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md) |

## Decision Questions

Answer these questions before choosing an overlay:

1. Which service or runner owns the apply authority?
2. Does the selected model apply an exact saved binary plan, or does it create
   a new run from the merged configuration?
3. Which state boundary, identity, and approval controls govern that model?

Do not combine profiles for the same state boundary. A platform can host a
runner-managed workflow without changing it into an HCP Terraform workspace.

## Then Choose an Overlay

Use the applicable overlay after selecting the profile:

* [GitHub Actions](../platform-overlays/github-actions.md)
* [Azure DevOps](../platform-overlays/azure-devops.md)
* [HCP Terraform](../platform-overlays/hcp-terraform.md)

## Route Connections

Return to the [bespoke module authoring route](../routes/bespoke-module-authoring.md),
[published module upgrade route](../routes/published-module-upgrade.md), or
[client handover route](../routes/client-handover.md) when recording the
selected delivery context.

## One State, One Apply Authority

[Execution profile] Assign one apply authority to each state boundary.

This is a state decision, not a repository decision.

Each root needs one active apply mechanism.

Do not let an HCP workspace and a CI runner apply the same state.

[Terraform universal] State identifies the managed objects for a root.

Two authorities create an ownership conflict.

## Compare Apply Inputs

| Profile | Review input | Apply input | After merge |
| --- | --- | --- | --- |
| Runner-managed | Binary-plan result | Selected artifact | Apply that artifact |
| HCP VCS workspace | Speculative run | Fresh normal run | Review normal run |

[Execution profile] These models are non-combinable for one state.

The runner model retains a selected plan through apply.

The HCP model creates a run for the merged commit.

> [!WARNING]
> Do not apply an HCP speculative pull-request plan after merge.
> Review the fresh normal run for the merged commit.

> [!WARNING]
> Do not replace a selected runner-managed plan with a fresh post-merge plan.
> Start a new reviewed plan cycle when replacement is needed.

## Selection Steps

1. Identify the root path.
2. Identify the state key or workspace.
3. Identify the service that applies changes.
4. Identify the apply input.
5. Confirm no second service can apply that state.
6. Identify review evidence.
7. Identify policy evidence.
8. Identify approval evidence.
9. Identify recovery ownership.
10. Record the profile.

## Scenario

The `payments` root stores `terraform.tfplan` in a CI artifact.

Select runner-managed reviewed plan for `payments-prod`.

The `networking` root maps to an HCP VCS workspace.

Select HCP Terraform VCS workspace for `networking-prod`.

```text
payments-prod: selected terraform.tfplan artifact
networking-prod: normal run for merged revision
```

## Decision Record

```text
root: roots/payments
state_identifier: payments-prod
execution_profile: runner-managed-reviewed-plan
apply_authority: configuration delivery runner
apply_input: build 912 / payments-plan
approval_evidence: platform approval record
policy_evidence: check result locator
recovery_owner: operations and platform owner
```

The input must identify a retrievable run or artifact.

A branch name is insufficient provenance.

## Failure Modes

| Failure | Consequence | Correction |
| --- | --- | --- |
| HCP and runner apply one state | Ownership conflict | Disable one path |
| HCP PR plan applied after merge | Wrong candidate | Locate normal run |
| Fresh runner main plan | Review changes | Retrieve artifact |
| Green plan treated as approval | Governance overstated | Record control |

## Verification

* Root and state are recorded
* One authority is recorded
* Apply input matches profile
* Run or artifact locator is retrievable
* Approval and policy locations are known
* Same-state serialization exists
* Destroy is separate
* Recovery owner is named

## Handover and Gaps

Record profile, authority, identity, evidence locator, and recovery owner in
the [handover evidence pack](../core/handover-evidence-pack.md).

[Gap] Backend, retention, recovery, and approval settings are policy-owned.