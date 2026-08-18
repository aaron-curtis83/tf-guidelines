---
title: Terraform State and Boundaries
description: Portable guidance for independently deployable Terraform roots and isolated state
---

## Choose Boundaries by Lifecycle

Create an independently deployable root when a capability has a distinct
owner, lifecycle, release cadence, security boundary, blast radius, or replacement requirement.
Keep related resources together only when they need one plan, apply, and recovery action.

[Terraform universal] Each root's state tracks its managed resource instances,
so a boundary affects addressing, locking, recovery, permissions, and scope.

## Keep State Isolated

Use separate state for independently operated roots. Do not use one shared
state merely because resources share a platform, subscription, account, or
repository. Make cross-boundary dependencies explicit through outputs, data sources, or external contracts.

Backend selection, encryption, retention, recovery, and access ownership
remain organization policy, not portable guidance in this library.

## Map Objects to State

[Terraform universal] State maps a Terraform resource instance address to one
remote object. A module path and collection key are part of that address.

This topology has two independently operated roots and two state boundaries.

```text
roots/
	network/
		main.tf
		outputs.tf
	application/
		main.tf
		data.tf
```

The network root can expose an output contract. The application root should use
that contract, a data source, or a documented external interface, rather than
the producer's internal resource address or local value.

```hcl
output "subnet_ids" {
	description = "Subnet identifiers for approved consumers."
	value = {
		app = azurerm_subnet.app.id
	}
}
```

[Training evidence] Locals are internal to their defining module. Outputs are
the values a root or module intentionally publishes for a consumer.

## Test a Proposed Boundary

Assess the actual operation owner for each question, not only the repository
team name.

| Boundary test | Keep together when | Separate when |
| --- | --- | --- |
| Apply cadence | Objects change in one release | Teams release them independently |
| Failure scope | One recovery action is required | Each capability can recover alone |
| Access | One operator identity needs both | Different permissions are required |
| Lifecycle | Retirement happens together | One capability outlives the other |
| Dependency | A direct resource dependency is needed | A stable output contract is sufficient |
| State risk | One lock and plan remain understandable | Combined plans hide unrelated changes |

Record the producing root, output or external identifier, consumer, data
classification, and change-notification expectation for every cross-root
dependency.

## Use Lock-Capable Remote Backends

[Terraform universal] A remote backend with locking capability can serialize
operations for a shared root. Locking reduces concurrent state-write risk; it
does not decide who may apply, approve, recover, or destroy.

For a team-operated root, record its backend, key or workspace, lock behavior,
access owner, and operation identity.

> [!IMPORTANT]
> Locking protects one state boundary. It does not create a transaction across
> roots. Coordinate cross-root releases explicitly.

[Assumption] A lock-capable remote backend is expected for shared production
roots. This library has not verified an organization-wide backend standard.

[Gap] Backend selection, encryption, access ownership, retention, backups,
and recovery objectives require organization policy.

## Configure a Backend Deliberately

A backend block belongs to the root configuration. Reusable child modules do
not configure a backend because the calling root owns state and initialization.

```hcl
terraform {
	backend "azurerm" {}
}
```

[Terraform universal] Backend blocks cannot refer to input variables, locals,
or named resource attributes. Provide environment-specific values through
approved initialization configuration rather than interpolation in the block.

A partial configuration permits approved values to be supplied at init time.

```bash
terraform init -backend-config=environment/dev.backend.hcl
```

Treat a backend configuration file as sensitive when it contains credentials or
classified identifiers. This example does not prescribe its storage mechanism.

## Initialize and Change Backends

Run initialization when first preparing a root, after provider or module source
changes, and after a backend configuration change.

```bash
terraform init
terraform init -reconfigure
```

[Terraform universal] Changing backend configuration requires `terraform init`.
Terraform can prompt to migrate existing state when appropriate. Review that
action before accepting it.

[Gap] Backend migration approval, state backup location, backup verification,
and recovery testing are organization-owned decisions. This page intentionally
does not provide an unverified rollback procedure.

## Shape Cross-Root Interfaces

Publish only values a consumer needs and document any output shape change.

```hcl
output "service_connection" {
	description = "Connection values for an approved consumer."
	value = {
		endpoint = "https://${azurerm_linux_web_app.api.default_hostname}"
		name     = azurerm_linux_web_app.api.name
	}
}
```

Prefer named object fields to positional output values. Do not expose provider
objects or complete state snapshots as an interface.

| Change | Consumer consequence |
| --- | --- |
| Add an output field | Usually additive when existing shape remains |
| Rename an output | Requires a coordinated consumer update |
| Change a value type | Requires compatibility review and tests |
| Remove an output | Requires a replacement or accepted breaking change |
| Add a sensitive output | Requires data-handling review |

Run a consumer plan against a proposed producer contract when it is safe and
practical. A clean producer plan does not establish consumer compatibility.

## Operate One Root at a Time

Name the root and state boundary in the operation record.

```bash
terraform fmt -check -recursive
terraform init
terraform validate
terraform plan -out=reviewed.tfplan
terraform show -no-color reviewed.tfplan
```

[Terraform universal] `terraform apply reviewed.tfplan` executes that saved
plan without another confirmation prompt. A saved plan can contain sensitive
data and becomes stale when relevant state, configuration, inputs, or provider
behavior changes.

## Recognize Boundary Failure Modes

| Symptom | Likely cause | First response |
| --- | --- | --- |
| One plan has unrelated changes | Roots were combined by layout | Reassess lifecycle and ownership |
| Apply waits on a lock | Another operation owns the state | Coordinate with its owner |
| Consumer fails after release | Output contract changed | Restore compatibility or coordinate update |
| Refactor plans create and destroy | Address mapping is absent | Stop and classify migration |
| Target plans a duplicate object | Ownership crossed incorrectly | Stop and review both roots |
| Recovery needs unknown access | Backend ownership is undocumented | Escalate to policy owner |

Do not force-unlock or manually alter state as a routine response. Those
actions require a backend-specific recovery process and accountable operator.

## Record Boundary Evidence

Record root identity, state identity, interfaces, operation authority, change
evidence, migration history, and the recovery escalation contact. Do not store
secret values; record retention and storage remain an organization assumption.

## Change Boundaries Deliberately

A boundary change can alter resource addresses and ownership of objects
already recorded in state. Assess the migration before changing module paths,
root layout, `for_each` keys, or resource names. Use the
[migration and refactoring guidance](migration-and-refactoring.md) for the
state transition.

## Route Connections

State boundaries are central to the
[bespoke module authoring route](../routes/bespoke-module-authoring.md), the
[published module upgrade route](../routes/published-module-upgrade.md), and
the [client handover route](../routes/client-handover.md).

## Related Core Guidance

Continue with [decision pathways](decision-pathways.md),
[testing and change assurance](testing-and-change-assurance.md), and the
[handover evidence pack](handover-evidence-pack.md).