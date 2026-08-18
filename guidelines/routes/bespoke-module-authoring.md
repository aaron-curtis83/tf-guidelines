---
title: Bespoke Module Authoring Route
description: Ordered guidance for defining and delivering a bespoke Terraform module
---

## Bespoke Module Authoring

Use this route when you are creating a module for a defined capability.

### Entry Criteria

Start when a capability has known intended consumers but its Terraform module
contract, root ownership, or delivery shape is not yet final. This route also
fits a private module that may later become reusable. It does not decide that a
module must be published to a registry.

Bring the following information to the first step:

* The capability outcome and the team or client accountable for it
* Expected consumers and the environment or root configurations they use
* A preliminary lifecycle, dependency, and operational ownership view
* Known provider, identity, state, or recovery constraints
* A place to record design decisions and delivery evidence

Local lesson sources on module structure and contracts support separating a
reusable capability from its root configuration. [Gap] The approved module
registry, support period, and publication process are not established here.

### Expected Decisions

| Decision | Record | Escalate when |
| --- | --- | --- |
| Module or root responsibility | Public module boundary and root-owned concerns | Consumers require incompatible lifecycle or access models |
| State boundary | Resources that share lifecycle and apply authority | Backend or recovery expectations are unknown |
| Contract shape | Inputs, outputs, provider requirements, and compatibility expectations | A shared contract changes without a named owner |
| Execution profile | One apply authority for the root state | More than one system may apply the same state |
| Assurance depth | Validation, tests, plan review, and operational checks | A credentialed integration test changes production-like infrastructure |

Do not resolve an unknown registry, backend, or approval baseline by copying a
template detail. Add it to the
[known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
register with an owner.

## Sequence

1. Complete scope discovery for the capability, ownership, lifecycle, and
   delivery context.
2. Use [decision pathways](../core/decision-pathways.md) before defining the
   module contract.
3. Continue through the [delivery lifecycle](../core/delivery-lifecycle.md).
4. Select an [execution profile](../execution-profiles/select-an-execution-profile.md)
   and then use the applicable platform overlay for delivery-specific controls.

### Step 1: Discover the Capability

Identify the result the capability provides, not only the first resource it
creates. Name its consumers, data dependencies, lifecycle events, and the
person or team that accepts operational responsibility.

| Discovery question | Example answer | Evidence or action |
| --- | --- | --- |
| What does the capability provide? | A private endpoint and DNS link for one application boundary | Draft a contract statement |
| Who consumes it? | Application roots in development and production | List consumers and their root paths |
| Who operates it? | Platform team until client handover | Record the accountable owner |
| What changes together? | Endpoint, DNS link, and approval flow | Test a candidate state boundary |
| What can fail independently? | Application deployment and shared networking | Consider separate root ownership |

[Practice recommendation] Capture a short decision record before writing the
module. The record makes a later contract or state-boundary review possible.
It is not a mandated documentation template.

### Step 2: Decide Before Defining the Contract

Use [decision pathways](../core/decision-pathways.md) to distinguish caller
inputs from internal calculations, a module from a root, and same-state work
from cross-state work. Then use
[state and boundaries](../core/state-and-boundaries.md) to test lifecycle,
ownership, blast radius, and apply-authority alignment.

| If the answer is yes | Choose | Why |
| --- | --- | --- |
| Multiple roots need the same capability with a stable contract | A module boundary | Callers need an explicit interface |
| The capability needs backend coordinates or provider configuration | Root ownership | Backend and provider configuration belong at the root |
| Consumers need a computed value but not implementation details | An output | Outputs define the cross-boundary contract |
| A child concerns only one cohesive package | A bundled nested module may be evaluated | The parent remains the public contract |
| Resources have different operators or change schedules | Separate roots or state boundaries | Independent operations require clear authority |

[Terraform universal] Provider configuration is owned by a root module. A
child module declares required providers and any aliases it accepts, but does
not establish the caller's provider configuration.

### Step 3: Move Through the Lifecycle

Continue through the [delivery lifecycle](../core/delivery-lifecycle.md) in
its stated order. The authoring route adds a contract focus to each stage.

1. Define typed inputs, internal locals, outputs, and provider requirements.
2. Write a consumer example that uses the public contract only.
3. Run formatting and validation appropriate to the repository.
4. Add plan-oriented contract tests before credentialed integration tests.
5. Review the infrastructure plan against the capability and state decision.
6. Record the release or source reference chosen for the consumer.
7. Retain evidence that links the accepted contract to the delivered revision.

### Original Worked Record

```text
capability: private-service-connectivity
public_inputs: subnet_id, private_dns_zone_id, environment
internal_values: name_prefix, normalized_tags
public_outputs: private_endpoint_id, private_ip_address
root_ownership: provider configuration and backend coordinates
state_boundary: application-connectivity-production
open_item: [Gap] module publication and support model
```

This record keeps a caller-visible contract separate from root configuration.
It does not prescribe a backend name, registry, or release model.

### Select a Profile and Overlay

Select the profile only after the root and state boundary are known.

| Delivery context | Follow | Then inspect |
| --- | --- | --- |
| A CI runner saves and applies a reviewed plan | [Runner-managed reviewed plan](../execution-profiles/runner-managed-reviewed-plan.md) | [GitHub Actions](../platform-overlays/github-actions.md) or [Azure DevOps](../platform-overlays/azure-devops.md) |
| A VCS-connected HCP Terraform workspace runs the root | [HCP Terraform VCS workspace](../execution-profiles/hcp-terraform-vcs-workspace.md) | [HCP Terraform](../platform-overlays/hcp-terraform.md) |

[Execution profile] A selected profile describes one apply authority for the
state boundary. Do not combine a saved-plan apply with an HCP Terraform normal
run for that same boundary.

### Authoring Failure Modes

| Failure mode | Reader-visible symptom | Corrective action |
| --- | --- | --- |
| Root concerns appear in the module contract | Callers supply backend or provider settings | Move root concerns out of the module interface |
| Contract includes an internal calculation as input | Callers repeat naming or tag logic | Replace the input with a local calculation |
| State boundary is assumed | Multiple teams can change the same objects | Complete the boundary decision before apply design |
| Template behavior becomes a policy claim | A local example is presented as required | Re-label as an example or open a gap |
| Nested child becomes an undocumented public API | Consumers reach into a packaged child | Keep the parent contract public or escalate release ownership |

### Verification Checklist

* The capability, consumers, owner, and lifecycle are recorded
* Module inputs, outputs, and provider requirements have a contract owner
* Root-owned backend and provider decisions are not exposed as module inputs
* The proposed state boundary has one apply authority
* Assurance includes a contract check and a plan review appropriate to risk
* The execution profile and platform overlay are selected after core decisions
* Unknown backend, registry, recovery, and retention decisions remain gaps

## Completion Condition

The module contract, delivery approach, and ownership decisions are clear
enough to begin implementation and validation.

### Completion Record

Use a concise record with links to the actual work artifacts:

```text
module: <module name>
contract_decision: <decision record location>
state_boundary: <root or workspace>
apply_authority: <selected profile>
assurance: <validation and plan review evidence>
open_gaps: <register IDs>
```

The route is complete when the record is reviewable and all unresolved policy
items have an owner. It does not require a resolved organization policy where
one has not yet been approved.

## Policy Escalation

Escalate unresolved backend, registry, retention, recovery, and handover
decisions through the [unresolved policy record](../index.md#unresolved-policy).

## Related Guidance

Continue with [modules and versioning](../core/modules-and-versioning.md),
[nested modules](../core/nested-modules.md) when a cohesive package may need a
child, and [testing and change assurance](../core/testing-and-change-assurance.md)
before implementation begins.