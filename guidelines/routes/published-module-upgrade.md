---
title: Published Module Upgrade Route
description: Ordered guidance for upgrading a consumed Terraform module safely
---

## Published Module Upgrade

Use this route when you are changing the version of a module that a
configuration already consumes.

### Entry Criteria

Start when a root configuration already calls a module and the source,
revision, version, provider requirement, or contract behavior may change.
Begin before editing the module reference when practical, because the upgrade
decision depends on consumer compatibility and resource-address effects.

Collect these facts:

* Current module source and version or Git reference
* Intended target source and version or Git reference
* Affected configuration roots, workspaces, and state boundaries
* Current module inputs, outputs, provider mappings, and dependent resources
* Release notes, migration notes, or source evidence available to the team
* Delivery owner and selected or proposed execution profile

[Gap] No organization-wide registry, support-window, or release-retention
policy is established by this library. Do not assume a versioning scheme is
approved merely because a module source currently uses it.

### Upgrade Decision Matrix

| Change signal | Review focus | Required record |
| --- | --- | --- |
| Version changes with unchanged interface | Release notes and plan impact | Compatibility assessment |
| Required input changes | Consumer configuration and validation behavior | Caller migration decision |
| Output changes | Downstream root and automation dependencies | Output consumer inventory |
| Provider requirement changes | Root provider version and alias mapping | Provider compatibility result |
| Resource address changes | State migration instructions | Same-state or cross-state classification |
| Source changes | Trust, release, and support ownership | Source-selection rationale |

[Terraform universal] A module upgrade can change planned infrastructure even
when the call syntax appears unchanged. Treat the plan and state-impact review
as part of the compatibility decision.

## Sequence

1. Complete a compatibility and state-impact pre-flight using
   [modules and versioning](../core/modules-and-versioning.md) and
   [state and boundaries](../core/state-and-boundaries.md).
2. Continue through the [delivery lifecycle](../core/delivery-lifecycle.md).
3. Select an [execution profile](../execution-profiles/select-an-execution-profile.md)
   and then use the applicable platform overlay for delivery-specific controls.

### Step 1: Complete Compatibility and State Pre-Flight

Read [modules and versioning](../core/modules-and-versioning.md) first, then
[state and boundaries](../core/state-and-boundaries.md). Build a before-and-
after inventory rather than relying on release labels alone.

| Inventory item | Before | After | Decision |
| --- | --- | --- | --- |
| Module source | Current source reference | Target source reference | Confirm ownership and expected trust model |
| Module version | Current version or ref | Target version or ref | Identify documented compatibility change |
| Inputs | Current caller values | New, removed, or changed values | Update caller or defer upgrade |
| Outputs | Consumed output names | Changed values or sensitivity | Update downstream consumers |
| Providers | Root mappings and constraints | Target requirements and aliases | Confirm root can satisfy them |
| Addresses | Existing state addresses | Expected planned addresses | Classify migration requirement |

[Terraform universal] If the target module changes managed-resource or module
addresses, use the migration guidance before approving an apply. A plan that
shows replacement or removal is evidence to investigate, not proof that a
destructive change is intended.

### Step 2: Interpret Version Evidence

Local lesson sources show that version selection can use a registry `version` constraint or a
Git `?ref=` reference depending on the selected module source. These are
source-specific mechanisms, not an organization mandate.

Use this short original evidence record:

```text
consumer_root: payments-production
module_call: module.observability
current: registry.example/observability 2.3.1
target: registry.example/observability 3.0.0
interface_change: alerting object replaces alert_email input
address_change: none declared in reviewed migration notes
decision: update caller object and require plan review
```

The record names the observed source format without asserting that the example
registry is required or that a major version always predicts exact plan impact.

### Step 3: Continue Through the Lifecycle

Use the [delivery lifecycle](../core/delivery-lifecycle.md) after the pre-
flight. For an upgrade, focus each stage on the consumer impact.

1. Confirm the owning team accepts the target source and version.
2. Update caller inputs, outputs, and provider mappings as needed.
3. Run contract, static, and targeted tests that can detect consumer changes.
4. Produce a plan from the proposed root configuration.
5. Compare planned adds, changes, replacements, and removes to the inventory.
6. Record any address migration, expected operational interruption, or rollback
   limitation before applying.
7. Retain the delivered revision, plan or run record, and outcome.

### Step 4: Select Profile and Overlay

Choose the profile after the upgrade plan can be associated with the affected
state boundary.

| Apply model | Upgrade delivery evidence | Overlay boundary |
| --- | --- | --- |
| Runner-managed reviewed plan | Reviewed plan artifact, selected run, and apply outcome | [GitHub Actions](../platform-overlays/github-actions.md) or [Azure DevOps](../platform-overlays/azure-devops.md) |
| HCP Terraform VCS workspace | Merged commit, normal run, policy result, and apply outcome | [HCP Terraform](../platform-overlays/hcp-terraform.md) |

[Execution profile] Do not re-plan after review in a runner-managed exact-plan
flow. [HCP Terraform] Do not apply a pull-request speculative plan after
merge; review the normal run created from the merged revision.

### Upgrade Failure Modes

| Failure mode | Why it happens | Response |
| --- | --- | --- |
| Release notes are treated as a complete migration plan | Consumer-specific values and state differ | Compare contract and plan to the actual root |
| New provider alias is not passed by the root | The module declares an alias but root mapping is absent | Correct the root provider mapping before plan review |
| Output consumer is missed | Output use exists outside the changed call site | Search configuration and delivery automation for use |
| Address change is accepted as incidental | A replacement or remove is not classified | Follow migration and refactoring guidance |
| Version pin is changed without source review | The source or trust boundary also changed | Record source-selection rationale and owner |
| Artifact or HCP run is not linked to the upgrade | Evidence cannot identify what was applied | Add the run or workspace record before handover |

### Verification Checklist

* Current and target source references are recorded
* Inputs, outputs, provider requirements, and aliases are compared
* State-address impact is classified before apply approval
* The plan or normal run is reviewed against the compatibility inventory
* Expected replacements, removals, and operational effects are explained
* The selected profile's evidence is linked to the merged or reviewed revision
* Registry, retention, recovery, and approval uncertainties remain gaps

## Completion Condition

The compatibility assessment, state impact, and delivery evidence support a
controlled upgrade decision.

### Completion Record

```text
root: <configuration root>
module: <module call>
from: <source and version>
to: <source and version>
compatibility: <accepted, deferred, or rejected>
state_impact: <none, same-state migration, or cross-state transition>
delivery_evidence: <plan artifact or HCP normal run>
open_gaps: <register IDs>
```

The upgrade route completes when the result is accepted, deferred, or rejected
with enough evidence for another reviewer to understand the decision.

## Policy Escalation

Escalate unresolved backend, registry, retention, recovery, and handover
decisions through the [unresolved policy record](../index.md#unresolved-policy).

## Related Guidance

Use [migration and refactoring](../core/migration-and-refactoring.md) for
address changes, [testing and change assurance](../core/testing-and-change-assurance.md)
for test selection, and [handover evidence pack](../core/handover-evidence-pack.md)
when the upgraded root moves to client operations.