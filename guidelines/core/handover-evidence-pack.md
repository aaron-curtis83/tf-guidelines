---
title: Terraform Handover Evidence Pack
description: Portable records for transferring Terraform operational responsibility
---

## Purpose and Boundary

Prepare an operator-usable evidence index for each transferred Terraform root. It points to authoritative records, not source copies, state, secrets, or a fabricated recovery procedure.

to avoid unintentionally changing management ownership.
[Terraform universal] State and configuration together determine the objects Terraform manages. A receiving operator needs both the root and state identity to avoid unintentionally changing management ownership.

[Practice recommendation] Keep the index concise enough for an incident but complete enough to identify ownership, evidence, dependencies, and open decisions.

Portable handover guidance does not select a backend, records system, retention
period, access model, recovery objective, or approval workflow. Record the
local arrangement and accountable owner as a fact or a gap.

## Handover Entry Conditions

Start handover only after the receiving operator has a named role and evidence
access. The delivery team remains accountable until the transfer decision occurs.

| Entry condition | Evidence needed | Failure gate |
| --- | --- | --- |
| Root identity | Repository path and revision | Root cannot be distinguished from a module |
| State boundary | Backend or workspace reference and owner | State access owner is unknown |
| Execution authority | Profile, platform, or workspace reference | More than one apply authority is assumed |
| Current delivery | Plan and apply or normal-run evidence | Revision cannot be tied to state |
| Dependencies | Outputs, consumers, and owners | A consumer may be broken by change or destroy |
| Recovery escalation | Contact and decision record | Operator cannot route an incident |

[Gap] Acceptance authority, retention, backup, recovery, and final handover
accountability remain organization-owned decisions. An index records these gaps;
it does not silently close them.

## Operator Evidence Index

Use this original record as an index. Replace placeholders with authorized
references; exclude credentials, state contents, plans, and sensitive outputs.

```text
capability: payments-api
root: roots/payments
configuration_revision: <commit or release reference>
state_identity: payments-production
state_location: <approved backend or workspace reference>
state_owner: <role or team>
module_inventory: example/payments-api/azurerm 2.4.0
execution_profile: runner-managed reviewed plan
executor: <run, pipeline, or workspace reference>
identity_boundary: <identity owner and access-decision reference>
test_evidence: <validation, test, and integration references>
plan_evidence: <reviewed plan or normal-run reference>
apply_evidence: <apply or normal-run outcome reference>
migration_history: <moved, removed, import, or none>
drift_record: <latest assessment or none>
recovery_contact: <operations and platform escalation reference>
open_decisions: DR-01 retention and recovery
reviewed: 2026-08-18
```

The record identifies control owners. It does not prove identity claims, RBAC,
reviewer approval, artifact signing, or a platform configuration baseline.

## Root, State, and Module Inventory

Identify the root Terraform evaluates and the state boundary that records its
managed resource instances. A module directory is not automatically a root.

| Record | Include | Operator question |
| --- | --- | --- |
| Capability | Purpose and service owner | What outcome does this root support? |
| Root | Repository path and working directory | Which configuration is safe to run? |
| State | Backend key, workspace, or approved locator | Which state does this root manage? |
| State owner | Accountable role and escalation contact | Who decides recovery actions? |
| Module inventory | Sources, versions, provider requirements | Which contracts constrain upgrades? |
| Dependencies | Producing root, output, consumer, owner | Who is affected by change or destroy? |

[Terraform universal] A state maps a resource instance address to a remote
object. Pair the root and state identity in every plan, apply, drift, migration,
and destruction record.

[Practice recommendation] Record module sources and selected revisions, but do
not claim a supported release window or registry policy until governance
approves one.

## Delivery Authority by Profile

The index identifies the selected profile without copying platform mechanics
into portable records.

| Profile | Record | Do not infer |
| --- | --- | --- |
| Runner-managed reviewed plan | Planning run, selected artifact, state, apply run, checksum when available | Approval, retention, or signing |
| HCP Terraform VCS workspace | Workspace, VCS revision, normal run, policy result, apply outcome | Auto-apply, edition, policy assignment, or permissions |

[Execution profile] A runner-managed profile applies a selected saved plan. Use
[runner-managed reviewed plan](../execution-profiles/runner-managed-reviewed-plan.md)
for its provenance procedure and the applicable platform overlay for mechanics.

[Execution profile] An HCP Terraform VCS workspace creates a fresh normal run
from the merged revision. It does not apply a pull-request speculative plan.
Use [HCP Terraform VCS workspace](../execution-profiles/hcp-terraform-vcs-workspace.md)
for workspace evidence fields.

## Plan, Apply, and Test Evidence

Capture enough information to connect an intended change to its execution. A
statement such as "applied successfully" is not an operational record.

| Evidence | Minimum index fields | Operator use |
| --- | --- | --- |
| Validation | Command or system result, revision, time | Establishes configuration check outcome |
| Test | Tier, result, fixture or environment class | Distinguishes mock from integration evidence |
| Plan | Root, state, revision, action summary, review reference | Explains intended transition |
| Apply | Executor, run or workspace, result, time | Connects execution to the plan or normal run |
| Verification | Focused plan or service observation | Records observed outcome |

[Terraform universal] A saved plan can contain sensitive values. Point to its
approved storage and handling class; do not paste plan content or sensitive
checksums into a broadly shared handover record.

For a normal root, command evidence can include:

```bash
terraform fmt -check -recursive
terraform init
terraform validate -no-color
terraform plan -out=reviewed.tfplan
terraform show -no-color reviewed.tfplan
```

Distinguish this portable sequence from the platform that created or applied the
authoritative plan.

## Drift, Migration, and Destroy Records

Record drift and failures separately. A plan can reveal a difference but cannot
decide whether configuration or remote infrastructure should change.

```text
event: drift assessment
root: roots/payments
state_identity: payments-production
observed_by: <plan or workspace run reference>
classification: expected change | unapproved change | provider read failure
decision_owner: <capability owner>
action: reconcile configuration | reviewed repair | investigate provider
evidence: <record reference>
```

| Event | Required record | Failure gate |
| --- | --- | --- |
| Drift | Address, classification, owner, decision | Scope is not understood |
| State lock | Active operation and executor | Owner cannot be identified |
| Same-state refactor | Old and new address, `moved`, plan | Unexpected replacement occurs |
| Cross-state transfer | Source release, target import, both states | More than one root owns the object |
| Destroy or replacement | Explicit plan, consumers, data impact, executor | Retirement or data disposition is unknown |

[Terraform universal] A `moved` block maps addresses within the evaluated
state. A cross-state transfer needs distinct source release and target import
operations. It is not one address move.

Do not use force-unlock or direct state editing as a handover runbook. These
actions require a backend-specific, organization-approved recovery procedure.

## Access and Escalation Matrix

Record roles and decision boundaries, not secrets or user accounts.

| Boundary | Accountable contact | Evidence reference |
| --- | --- | --- |
| Capability behavior | Service or product owner | Scope and ownership record |
| Root changes | Configuration owner | Repository and review reference |
| State recovery | Platform or operations owner | Backend or workspace decision |
| Apply authority | Delivery or workspace owner | Selected profile record |
| Identity and permissions | Identity or security owner | Access-decision reference |
| Consumer contract | Producing and consuming owners | Output and dependency record |

[Assumption] A receiving operator can contact each role through an approved
support channel. Validate this before declaring handover complete.

## Handover Review Checklist

* Root, revision, and state identify one deployable boundary
* Module inventory and contracts identify producers and consumers
* Selected profile points to the authoritative plan and apply evidence
* Tests distinguish validation, mock, and integration scope
* Drift, migration, replacement, and destroy history are indexed when present
* State, identity, approval, and recovery decision owners are named
* Sensitive values, state contents, and unapproved recovery steps are absent
* DR-01 gaps remain visible with their accountable organization owner

## Preserve Open Decisions

[Gap] The organization has not established universal requirements for backend
choice, evidence retention, approval, backups, recovery, registry ownership,
or handover acceptance. Reference the
[known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
register with the affected root and decision owner.

An index may report a local arrangement. It must not turn that arrangement into
organization policy or claim a platform setting was independently verified when
the evidence only shows workflow source.

## Route Connections

Follow the [client handover route](../routes/client-handover.md) for transfer.
Use the [bespoke module authoring route](../routes/bespoke-module-authoring.md)
and [published module upgrade route](../routes/published-module-upgrade.md) to
collect evidence during delivery rather than reconstructing it at handover.

## Related Core Guidance

Review [delivery lifecycle](delivery-lifecycle.md),
[state and boundaries](state-and-boundaries.md),
[testing and change assurance](testing-and-change-assurance.md), and
[migration and refactoring](migration-and-refactoring.md).