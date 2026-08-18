---
title: Terraform Delivery Lifecycle
description: Portable lifecycle guidance for planning, delivering, operating, and retiring Terraform configurations
---

## Purpose and Scope

Use this lifecycle for each independently deployable Terraform root. It links capability ownership, contracts, state, evidence, operations, and retirement without selecting a backend, CI platform, or approval mechanism.

[Terraform universal] Terraform configuration expresses the intended state of managed infrastructure. Terraform state records the resource-instance addresses and remote objects Terraform manages. A lifecycle decision must therefore account for both configuration and its state boundary.

[Practice recommendation] Assign accountable roles for the capability, root, state, execution authority, and receiving operator before apply. One team can hold several roles, but a handoff must name them.

Portable guidance ends at the execution-profile boundary. Use the selected
[execution profile](../execution-profiles/select-an-execution-profile.md) for
runner, workspace, approval, identity, and artifact mechanics.

## Accountable Stages

| Stage | Accountable role | Inputs | Required output | Failure gate |
| --- | --- | --- | --- | --- |
| Discover | Capability owner | Need, consumers, constraints | Scope and operational owner | Scope or owner is unknown |
| Design | Configuration owner | Scope, dependencies, consumers | Root, module, and state design | Boundary or contract is unclear |
| Prepare | State and delivery owner | Root design, backend decision | Initialized execution context | State owner or executor is unknown |
| Assure | Change owner | Revision, inputs, test scope | Reviewed plan and test evidence | Unexplained plan action or failed check |
| Deliver | Approved executor | Reviewed change and profile record | Apply outcome and verification | Evidence cannot be tied to the state |
| Operate | Service operator | Root, state, contacts, run history | Drift and incident decisions | Ownership or recovery contact is absent |
| Retire | Capability and state owner | Retirement decision and dependencies | Controlled transition or destroy record | Consumers or data disposition are unknown |

[Practice recommendation] The accountable role may delegate execution, but it
retains responsibility for the stage output and unresolved decisions.

## Discover the Capability

Identify what the root manages, its consumers, expected operator, and whether it changes or retires independently of neighboring infrastructure.

Use a separate root when lifecycle, access, blast radius, or recovery ownership differs. See [state and boundaries](state-and-boundaries.md) for the boundary tests and cross-root interface rules.

> [!WARNING]
> A shared repository or subscription does not establish a shared lifecycle or
> state boundary. Do not combine roots only because their resources are nearby.

## Design Contracts and Boundaries

Name the root, state coordinate, module sources, output interfaces, and execution profile in the delivery record before recording objects in state.

[Terraform universal] A resource address includes its resource path, module
path, and collection key. Classify a planned address change before implementation.

[Practice recommendation] Keep provider configuration and backend configuration
in the root. Reusable child modules declare requirements and accepted aliases
but do not own state.

## Prepare the Execution Context

Preparation makes the root reproducible for the authorized executor. It does not authorize an apply.

Run a normal root sequence only after obtaining approved backend configuration
through the selected local process:

```bash
terraform fmt -check -recursive
terraform init
terraform validate -no-color
terraform plan -out=reviewed.tfplan
terraform show -no-color reviewed.tfplan
```

[Terraform universal] `terraform plan -out=FILE` creates a saved plan and `terraform apply FILE` executes that plan without a new confirmation prompt. Saved plans can contain configuration, input values, and sensitive data.

Record the Terraform version, provider selections, root, state, and command
outcome. Do not copy secret values into a plan or handover record.

> [!IMPORTANT]
> A saved plan is a proposed transition. It does not prove approval, policy
> compliance, or that it remains current.

## Assure the Proposed Change

Perform checks that match the change class. Add focused test, consumer, integration, or migration evidence when a contract, provider behavior, or state address changes. Stop when any change proposes unexplained destruction, replacement, duplicate creation, or unverified consumer impact.

Use [testing and change assurance](testing-and-change-assurance.md) for test
tier selection. Use [migration and refactoring](migration-and-refactoring.md)
for `moved`, `removed`, and `import` procedures.

## Review Plan Evidence

Review the rendered plan with its revision, module versions, root, state
coordinate, action summary, dependency or migration context, and external test
or approval references.

[Terraform universal] A plan compares configuration with available state and
provider information at that point in time. It can expose drift and replacement
but does not execute a repair.

Stop the lifecycle when the plan proposes an unexplained destroy, duplicate
creation, replacement, import, address move, or change outside the agreed
scope. Correct the configuration or reassess the boundary before continuing.

## Deliver Through One Apply Authority

[Practice recommendation] Choose one apply authority for one state boundary at
a time. Coordinate all other operators with that authority and do not create a
second apply path that can race the same state.

The delivery record must state which profile applies:

* A [runner-managed reviewed plan](../execution-profiles/runner-managed-reviewed-plan.md)
  applies a selected saved plan through CI.
* An [HCP Terraform VCS workspace](../execution-profiles/hcp-terraform-vcs-workspace.md)
  applies a normal run from the merged revision.

[Execution profile] The selected profile owns its platform-specific run,
approval, identity, and evidence mechanics. Do not attribute those mechanics to
Terraform universal behavior.

After apply, capture the apply outcome, executor identity reference, run or
workspace reference, and observable verification result. Run a focused plan or
service check when it is appropriate and safe.

## Respond to Drift

Treat drift as an observed difference requiring classification, not as a reason
to run an unreviewed apply. Start with a plan from the intended root and state.

```bash
terraform init
terraform plan -out=drift-review.tfplan
terraform show -no-color drift-review.tfplan
```

For expected changes, record an owner before reconciling configuration. For
unapproved remote changes, missing objects, locks, or provider read failures,
pause and classify ownership, dependency impact, and the active operation.
Do not run a broad repair, recreate an object, force-unlock, or treat an
incomplete plan as evidence that no drift exists.

[Gold-standard example] Internal GitHub workflows separate plan and drift
identities and serialize state-key work. This is an observed pattern, not an
organization identity, concurrency, or approval baseline.

## Respond to Destroy and Replacement

Destruction and replacement are lifecycle events, not generic cleanup. Identify
the root, affected object addresses, consumers, data disposition, executor,
and recovery contact before requesting an operation.

Use an explicit plan for the retirement or replacement and preserve its review
evidence. A generic example is:

```bash
terraform plan -destroy -out=retirement.tfplan
terraform show -no-color retirement.tfplan
```

[Gap] Retention, destruction approval, backup, recovery objective, and break-
glass procedure require organization policy. Record the decision owner and
interim boundary rather than inventing a procedure.

## Worked Capability Scenario

This original record illustrates a portable payments API lifecycle. Its values
are placeholders and do not select a backend, identity, or approval model.

```text
capability: payments-api
root: roots/payments
state_identity: payments-production
state_owner: platform operations role
module: example/payments-api/azurerm 2.4.0
consumer_contract: api_endpoint and workload_identity_id
execution_profile: runner-managed reviewed plan
change: add private endpoint output
tests: format, validate, plan-mode contract test, consumer plan
plan_evidence: review run reference and reviewed.tfplan checksum
apply_evidence: selected-run reference and post-apply endpoint check
drift_response: owner classifies observed change before repair
retirement_decision: not approved; consumer and data review required
```

The operator must be able to locate each referenced record independently.

## Collect Lifecycle Evidence

Collect scope, boundary, initialization, test, plan, apply, drift, migration,
and retirement records during their lifecycle stage. Their authoritative
location can be a repository, delivery system, workspace, or approved records
system. Use the [handover evidence pack](handover-evidence-pack.md) as the
operator index for these pointers, owners, and open decisions.

## Open Policy Decisions

[Gap] The library does not establish organization-wide backend, retention,
approval, backup, recovery, registry, or handover-accountability policy. Name
the decision owner and current local arrangement without calling it universal.

Use the [known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
register when the missing decision blocks a delivery or handover outcome.

## Route Connections

* Start a new capability through the
  [bespoke module authoring route](../routes/bespoke-module-authoring.md)
* Assess a dependency transition through the
  [published module upgrade route](../routes/published-module-upgrade.md)
* Transfer operational responsibility through the
  [client handover route](../routes/client-handover.md)

## Related Core Guidance

Continue with [decision pathways](decision-pathways.md),
[state and boundaries](state-and-boundaries.md),
[testing and change assurance](testing-and-change-assurance.md), and
[migration and refactoring](migration-and-refactoring.md).