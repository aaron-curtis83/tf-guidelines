---
title: Client Handover Route
description: Ordered guidance for preparing Terraform delivery evidence for client operators
---

## Client Handover

Use this route when transferring operational responsibility for a Terraform
configuration to client operators.

### Entry Criteria

Start when a client team will operate, approve, investigate, or recover a
configuration root after delivery. Handover can begin before every policy gap
is resolved, but it cannot hide those gaps. Record the current evidence and
the owner required to close each missing decision.

Gather the following before the first review:

* Configuration-root paths and module source or version references
* State boundary or HCP Terraform workspace identifiers
* Current apply authority and selected execution profile
* Delivery, test, plan or run, apply, drift, and migration evidence locations
* Named operational contacts, access owners, and escalation channels
* Known backend, recovery, retention, policy, or identity uncertainties

[Gap] This library does not establish a universal state-recovery procedure,
handover accountability model, or artifact-retention period. Transfer the
known facts and escalate the missing policy to its owner.

### Handover Audience Matrix

| Reader | Needs to answer | Evidence to locate |
| --- | --- | --- |
| Client operator | What root, state, and dependencies am I operating? | Root inventory, boundary decision, dependency map |
| Delivery owner | What revision and checks were delivered? | Source reference, tests, plan or run, apply result |
| Platform owner | Who can execute and recover the delivery path? | Profile selection, identity owner, access and escalation record |
| Module consumer | Which contract and version are in use? | Module source, version, inputs, outputs, upgrade history |

The records can reside in the delivery team's selected systems. The route does
not prescribe an issue tracker, document repository, or service-management
tool.

## Sequence

1. Start with the [delivery lifecycle](../core/delivery-lifecycle.md) and
   [handover evidence pack](../core/handover-evidence-pack.md).
2. Use [decision pathways](../core/decision-pathways.md) when an ownership,
   state, or module question requires resolution.
3. Select an [execution profile](../execution-profiles/select-an-execution-profile.md)
   and then use the applicable platform overlay for delivery-specific controls.

### Step 1: Start With Lifecycle and Evidence

Follow [delivery lifecycle](../core/delivery-lifecycle.md) to connect
discovery, contract, state, delivery, operation, and retirement. Then create
the inventory defined by the
[handover evidence pack](../core/handover-evidence-pack.md).

| Evidence category | Operator question | Handover record |
| --- | --- | --- |
| Configuration | What directory is executable and what modules does it call? | Root path, source references, committed revision |
| State or workspace | What resources are managed together? | State boundary, workspace when applicable, accountable owner |
| Delivery | How did the reviewed change become applied? | Profile, run or artifact location, apply outcome |
| Assurance | What was tested and what remains untested? | Validation, test, plan-review, and integration evidence |
| Operations | How are drift, incidents, and planned changes handled? | Contacts, escalation route, known limitations |
| Migration | What address or state transitions already occurred? | Migration record, date, and remaining compatibility notes |

[Practice recommendation] Prefer links to immutable or durable delivery
records when available. This is an evidence-quality preference, not a defined
retention or storage policy.

### Step 2: Resolve Ownership Questions

Use [decision pathways](../core/decision-pathways.md) for ownership, module,
and state questions. The handover must distinguish facts from open decisions.

| Question | Record as confirmed when | Record as a gap when |
| --- | --- | --- |
| Who owns state operation? | A named team accepts responsibility | Ownership is informal or disputed |
| Who may apply? | One profile and executor are identified | Multiple systems may apply the same boundary |
| Who owns provider and identity access? | Root and access owner are named | Identity claims, RBAC, or recovery access are unknown |
| Who supports the module contract? | Source and version support contact is known | Registry or support policy is unavailable |
| How is recovery performed? | An approved procedure and owner are linked | Only assumptions or local experience exist |

Do not convert a current person, local template, or platform setting into a
permanent handover rule without ownership approval.

### Step 3: Select Profile and Overlay

Record the profile that controls future operations. It changes what evidence a
client operator expects during a normal change.

| Profile | Operator expects | Use this overlay |
| --- | --- | --- |
| Runner-managed reviewed plan | Reviewed saved-plan artifact, selected runner execution, and apply outcome | [GitHub Actions](../platform-overlays/github-actions.md) or [Azure DevOps](../platform-overlays/azure-devops.md) |
| HCP Terraform VCS workspace | Workspace, merged commit, normal run, policy result, and apply outcome | [HCP Terraform](../platform-overlays/hcp-terraform.md) |

[Execution profile] A state boundary has one apply authority. The handover
record must identify it, even if the operator does not have permission to
execute changes directly.

### Original Handover Evidence Record

```text
root: payments-production
state_boundary: payments-production
module_sources: observability 3.0.0; networking 1.8.2
profile: hcp-terraform-vcs-workspace
delivery_evidence: workspace normal run <run identifier>
assurance: validation and plan review linked; integration-test scope recorded
operations_owner: client platform team
open_items: [Gap] recovery procedure; [Gap] policy-set assignment baseline
```

This original record transfers current facts without inventing a workspace
naming rule, retention setting, or recovery policy.

### Handover Failure Modes

| Failure mode | Operator impact | Corrective action |
| --- | --- | --- |
| Only source code is transferred | State, execution, and evidence context is lost | Add root, profile, and delivery records |
| State owner is unnamed | Incidents cannot be routed | Record an owner or raise DR-01-equivalent gap |
| A speculative plan is presented as applied evidence | HCP run semantics are misrepresented | Link the merged normal run instead |
| A generic CI artifact is used without provenance | Runner-managed apply cannot be reconstructed | Link selected validation run and artifact details |
| Known recovery uncertainty is omitted | Client assumes an unsupported process exists | Keep the gap visible with an owner |
| Module source and version are absent | Future upgrades cannot begin safely | Add contract and upgrade inventory |

### Verification Checklist

* Each configuration root and state boundary has a named owner or a gap record
* Module sources, versions, public contracts, and dependent roots are listed
* The selected profile and appropriate overlay are linked
* Delivery evidence identifies a reviewed plan artifact or HCP normal run
* Test, plan-review, apply, drift, and migration evidence is discoverable
* Current access, identity, and escalation contacts are recorded
* Unresolved backend, recovery, retention, and policy decisions remain gaps

## Completion Condition

Client operators have the delivery evidence, execution context, and ownership
decisions needed to assume responsibility.

### Completion Record

```text
handover_scope: <root or application boundary>
operator_owner: <team or named role>
state_or_workspace: <identifier>
apply_authority: <profile and platform>
evidence_index: <location>
accepted_on: <date or pending>
open_gaps: <register IDs and owners>
```

The route completes when client operators can locate evidence and identify the
owner for every unresolved operational decision. It does not require inventing
missing policy content.

## Policy Escalation

Escalate unresolved backend, registry, retention, recovery, and handover
decisions through the [unresolved policy record](../index.md#unresolved-policy).

## Related Guidance

Review [state and boundaries](../core/state-and-boundaries.md) for state
ownership, [migration and refactoring](../core/migration-and-refactoring.md)
for transition history, and [known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
before declaring the handover ready.