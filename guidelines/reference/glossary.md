---
title: Terraform Practice Glossary
description: Shared terminology used across the Terraform Practice Guidelines
---

## Terms

| Term                      | Definition                                                                                       |
|---------------------------|--------------------------------------------------------------------------------------------------|
| Apply authority           | The selected runner or HCP Terraform workspace that is allowed to apply changes for one state boundary |
| Backend                   | Terraform configuration that determines where state is stored and how Terraform accesses it      |
| Configuration root        | A directly executable Terraform directory with its own state boundary                            |
| Delivery evidence         | Records that connect a delivered revision to validation, review, plan, apply, and approval outcomes |
| Execution profile         | The delivery model that defines how Terraform plans and applies changes                          |
| Gold-standard example     | An observed internal implementation that informs guidance without becoming a universal rule      |
| HCP Terraform normal run  | The fresh run created from a merged VCS revision, distinct from a pull-request speculative plan  |
| Independently deployable root | A configuration root with its own lifecycle, state, ownership, and apply authority            |
| Plan artifact             | A retained binary plan and associated evidence produced by a runner-managed workflow             |
| Practice recommendation   | Guidance derived from evidence that requires governance approval before becoming mandatory       |
| Saved plan                | The binary output from `terraform plan -out` that Terraform can later apply                      |
| Speculative plan          | A non-applying plan, commonly created for pull-request review                                    |
| State boundary            | The scope of resource addresses and ownership tracked together in one Terraform state            |
| Template contract         | The verified workflow version, inputs, secret names, and behavior required to adopt a template   |
| Workspace                 | An HCP Terraform unit that owns state, variables, run history, and configured execution behavior |

## Term Boundaries

Terms in this glossary describe concepts used by the library. They do not
choose products, permissions, approval levels, ownership assignments, or
retention periods. Read the accompanying evidence label before treating a
definition as behavior that applies to a particular platform.

| Do not conflate | With | Why the distinction matters |
| --- | --- | --- |
| Configuration root | Module | A root is executable and owns its state; a module exposes a reusable contract |
| State boundary | Repository boundary | One repository can contain multiple independently deployable roots |
| Apply authority | Code owner | A code owner can review code without being able to apply state |
| Saved plan | Speculative plan | A saved plan can be applied by Terraform; an HCP speculative plan is review-only |
| Platform overlay | Execution profile | An overlay names platform controls; a profile defines plan-to-apply semantics |
| Gold-standard example | Approved policy | An observed pattern needs independent governance approval to become mandatory |
| Assumption | Gap | An assumption awaits validation; a gap lacks sufficient evidence for a rule |

## Detailed Definitions

### Apply Authority

[Execution profile] The service, runner, or workspace allowed to apply changes
for one state boundary. It is selected after the configuration's state and
delivery ownership are understood. One state boundary must not have competing
apply authorities.

The term does not imply that the authority has a particular identity, approval
setting, break-glass process, or retention configuration. Those details belong
to the selected profile, platform overlay, or an approved policy.

### Backend

[Terraform universal] A backend specifies where Terraform stores state and how
Terraform accesses it. Backend configuration has Terraform-specific
constraints, including that it cannot reference variables or locals. A backend
does not by itself establish the organization-approved encryption, access,
locking, recovery, or retention model.

See [backend configuration](https://developer.hashicorp.com/terraform/language/backend)
for portable behavior and [known gaps and assumptions](known-gaps-and-assumptions.md)
for the unapproved organization decision boundary.

### Configuration Root

[Terraform universal] A configuration root is the directory executed as a
Terraform configuration. It owns root-level concerns such as backend and
provider configuration. A root can call modules, but it is not interchangeable
with the module package it consumes.

When one repository has several executable directories, identify each root and
its state boundary separately in delivery and handover records.

### Delivery Evidence

Delivery evidence links a reviewed change to the checks, plan or run, apply,
and operational outcome that explain it. The exact artifact, run-history, or
work-item system is platform and organization specific.

For a runner-managed profile, evidence may include the selected saved-plan
artifact and its validation run. For HCP Terraform, evidence includes the
merged commit's normal run, policy result, and apply status. Neither pattern
defines a universal retention or approval setting.

### Execution Profile

[Execution profile] A profile defines the relation between review and apply.
The library contains runner-managed reviewed-plan and HCP Terraform VCS
workspace profiles. Select one profile for a state boundary before applying a
platform overlay.

The profile does not choose GitHub Actions, Azure DevOps, a workspace naming
scheme, or a credential strategy. Those are overlay or policy concerns.

### Gold-Standard Example

[Gold-standard example] An observed internal implementation pattern. It is
useful for testing whether a design has considered a real delivery concern,
such as plan provenance. It does not prove that all controls were configured,
that every repository uses the pattern, or that the pattern has governance
approval.

### HCP Terraform Normal Run

[HCP Terraform] The remote run created for the merged revision tracked by a
VCS-connected workspace. It is distinct from a pull-request speculative plan.
The normal run is the relevant record for an applied merged change, subject to
the workspace's configured apply mode and permissions.

### Independently Deployable Root

An executable configuration root with a distinct lifecycle, state boundary,
operational owner, and apply authority. Use this term when deciding whether a
capability should be combined with or separated from another root.

Separate roots may still exchange carefully defined outputs or share a module.
They should not silently share an apply authority or assume identical recovery
responsibility.

### Plan Artifact

[Execution profile] A binary Terraform plan and associated evidence retained
by a runner-managed delivery flow. `terraform plan -out=FILE` creates an
artifact that `terraform apply FILE` can execute. It can contain sensitive
information and must be handled according to the delivery system's controls.

A plan artifact is not synonymous with every platform's plan display. HCP
Terraform speculative and normal runs use workspace run behavior instead.

### Practice Recommendation

[Practice recommendation] Guidance inferred from available evidence that would
benefit from governance approval before it becomes mandatory. It can guide a
local design discussion, but it must retain its label and owner until approved.

### Saved Plan

[Terraform universal] The binary output created by `terraform plan -out`. It
contains the full configuration, input values, and potentially cleartext
sensitive data. Applying it avoids creating a fresh plan, but does not itself
prove integrity, retention, or approval behavior.

### Speculative Plan

[HCP Terraform] A non-applying plan produced for review, commonly from a pull
request in a VCS-connected workspace. It is not a saved runner-managed plan
artifact and must not be presented as the object applied after merge.

### State Boundary

[Terraform universal] The set of resource addresses tracked together in one
Terraform state. State connects configuration addresses to managed objects.
Lifecycle, ownership, blast radius, dependency direction, and apply authority
help determine whether resources belong in one boundary.

The term does not select a backend implementation or recovery procedure.

### Template Contract

The verified inputs, secrets, outputs, versions, and behavior required to use
a reusable delivery template. A README and executable callers can disagree;
when they do, the difference is a gap until the template owner resolves it.

### Workspace

[HCP Terraform] A workspace owns a resource collection's state, variables,
run history, and remote operations. Workspace naming, policy-set assignment,
approval settings, edition, and retention require separate organization
decisions unless independently verified.

## Original Usage Examples

| Statement | Correct term | Reason |
| --- | --- | --- |
| "The application root and networking root can be applied separately" | Independently deployable roots | They have different state and execution boundaries |
| "The PR review is not the record of the merged deployment" | HCP Terraform normal run | The merge creates a fresh normal run |
| "The pipeline applies the reviewed binary file" | Saved plan and plan artifact | The runner-managed profile uses a selected plan artifact |
| "The provider alias is accepted by the module and configured by the root" | Module contract and configuration root | Provider configuration remains root-owned |
| "We do not know the state recovery method" | Gap | No verified policy supports a requirement |

## Glossary Verification

* Use [execution profiles](../execution-profiles/select-an-execution-profile.md)
	to select apply semantics before naming delivery evidence
* Use [evidence catalog](evidence-catalog.md) when a term carries a label
* Use [known gaps and assumptions](known-gaps-and-assumptions.md) before
	treating a platform setting or local pattern as policy
* Use [state and boundaries](../core/state-and-boundaries.md) when a root,
	module, or state-boundary term needs a design decision

## Related Reference

Use the [evidence catalog](evidence-catalog.md) to identify a term's claim
boundary. Use [known gaps and assumptions](known-gaps-and-assumptions.md) when
a term depends on unresolved organization policy.

Return to [the library index](../index.md) to select a route after resolving
the terms needed for the current task.