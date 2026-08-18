---
title: Select an Execution Profile
description: Choose the Terraform execution authority before following platform-specific delivery guidance
---

## Choose One Apply Authority

Select one profile for each independently deployable Terraform root before
following a platform overlay. The profile defines the relationship between
review evidence, the candidate that can change infrastructure, and the record
that proves what ran. It does not select a backend, approval rule, retention
period, or identity implementation.

[Terraform universal] A root configuration evaluates from its working
directory against one state. Keep the state identifier and its apply authority
together when making this decision.

| Delivery model | Select this profile | Candidate that can apply |
| --- | --- | --- |
| A runner saves a binary plan and later applies that selected file | [Runner-managed reviewed plan](runner-managed-reviewed-plan.md) | The selected saved plan |
| An HCP Terraform workspace is VCS-connected and runs the configuration | [HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md) | The HCP normal run for the merged revision |

[Execution profile] The selected profile applies to one state boundary, not to
an entire repository. Different roots in a monorepo can use different profiles
when their state, owners, and delivery evidence remain distinct.

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

## Non-Combinable Controls

[Execution profile] Assign one apply authority to each state boundary. This is
a state decision, not a repository decision. Do not let an HCP workspace and a
CI runner apply the same state.

> [!WARNING]
> Do not apply an HCP speculative pull-request plan after merge. Review the
> fresh normal run for the merged commit, which is a different candidate.

> [!WARNING]
> Do not replace a selected runner-managed plan with a fresh post-merge plan.
> Start a new reviewed plan cycle when replacement is needed.

| Incompatible pairing | Why it fails | Required correction |
| --- | --- | --- |
| Runner apply and workspace apply for one state | Two services can change one object mapping | Select one authority and retire the other path |
| HCP speculative run and post-merge HCP apply | The speculative run reflects PR head and plan-time state | Locate and assess the normal run for the merged commit |
| Selected saved plan and fresh post-merge runner plan | The second plan has different provenance and may propose different actions | Start a new plan, review, and decision cycle |
| Branch name and an apply candidate | A branch is mutable and does not identify the evaluated input | Record commit plus artifact checksum or HCP run ID |

## Required Selection Evidence

Create or update a profile decision before allowing an apply-capable path to
run. Store locators rather than copying plans, state contents, credentials, or
sensitive variable values into the record.

| Evidence | Why it is needed | Minimum content |
| --- | --- | --- |
| Root identity | Prevents a module or sibling root from being selected | Repository, root path, revision |
| State identity | Binds delivery to its managed object set | Backend or workspace locator and owner |
| Apply authority | Exposes the one service allowed to change this state | Runner identity or HCP workspace |
| Apply candidate | Lets an operator retrieve the actual delivery input | Artifact manifest or normal-run ID |
| Review evidence | Separates review from execution mechanics | PR, review, or plan-view locator |
| Policy and approval evidence | Avoids treating a successful plan as authorization | Configured control result or gap |
| Recovery contact | Routes failed or blocked work without inventing a procedure | Accountable role and escalation reference |

[Practice recommendation] Require all seven fields before classifying a root as
ready for an apply. This recommendation does not establish an organization-wide
approval, retention, or backend standard.

[Gap] Backend choice, artifact and run retention, recovery procedure, approval
requirements, and HCP Terraform edition are organization decisions. Link the
applicable root to [known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
when they are unresolved.

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