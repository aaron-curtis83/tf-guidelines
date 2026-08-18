---
title: Terraform Migration and Refactoring
description: Portable guidance for preserving managed-object identity during Terraform refactoring
---

## Treat Addresses as Operational Interfaces

Refactoring a Terraform configuration can change a resource or module address
without changing the intended real-world object. Assess every address change
before delivery to prevent Terraform from interpreting a move as a destroy and
create operation.

[Terraform universal] A `moved` block declares that Terraform should treat an
old address as the same object at a new address. The move is evaluated as part
of planning and supports refactoring while preserving the tracked object.

## Plan a Migration

1. Identify the current address, the target address, and the expected object
   identity.
2. Add and review the required `moved` blocks before removing the old address.
3. Validate and plan from a representative state for each supported upgrade
   path.
4. Record prerequisites, expected changes, rollback limits, and ownership.
5. Release migration instructions with the configuration or module change.

## Do Not Hide Destructive Changes

A `moved` block does not make every change non-destructive. Changes to resource
type, immutable attributes, provider settings, state boundaries, or object
ownership can still require replacement or a separately approved migration.
Escalate backend recovery and live-operation decisions because no organization
recovery policy is established here.

## Classify the Change First

| Change | State boundary | Mechanism |
| --- | --- | --- |
| Rename resource or module | Same state | `moved` |
| Change collection keys | Same state | One `moved` declaration per instance |
| Change immutable arguments | Same or new state | Replacement plan |
| Transfer ownership to another root | Cross state | Source `removed`, target `import` |
| Stop managing an object | Source state | `removed` |
| Adopt an existing object | Target state | `import` |

Do not use `moved` for a transfer between state boundaries. It does not move
objects, backend settings, locks, or history between states.

## Prepare a Migration Record

Record the change class, source and target address, state boundary, verified
provider-specific object ID, dependencies, expected plan, executor, evidence,
and cleanup decision. Use the address from `terraform state list`, not memory.

```bash
terraform init
terraform validate
terraform state list
terraform state show module.app.azurerm_linux_web_app.this
terraform plan
```

Stop if a preflight plan already proposes unexplained destruction, duplicate
creation, or replacement. Resolve the configuration mismatch before declaring
a migration.

## Use Same-State Moved Blocks

[Terraform universal] A same-state move keeps ownership in the current state
and changes the address Terraform uses for the same object.

```hcl
module "application" {
   source = "./modules/application"
}

moved {
   from = module.app
   to   = module.application
}
```

The example maps one module address inside one state. It does not migrate a
backend or transfer the module's objects to another root.

1. Add the destination address with the intended resource arguments.
2. Add the `moved` block in the configuration evaluated by the existing state.
3. Run formatting, validation, and a plan.
4. Review a move without unexpected create or destroy for the preserved object.
5. Apply through the selected execution profile and inspect the new address.

```bash
terraform fmt -check
terraform validate
terraform plan -out=address-move.tfplan
terraform show -no-color address-move.tfplan
```

For a collection refactor, map every old address to its intended key. Do not
assume source ordering establishes object identity.

| Existing address | Target address | Evidence |
| --- | --- | --- |
| `azurerm_subnet.this[0]` | `azurerm_subnet.this["app"]` | Same subnet identifier |
| `azurerm_subnet.this[1]` | `azurerm_subnet.this["data"]` | Same subnet identifier |

[Assumption] Keep a `moved` block for supported upgrade paths. Its retention
period is an organization compatibility-policy gap.

## Transfer Objects Across States

A cross-state transition changes ownership from one root and state to another.
It is not a `moved`-block operation.

[Training evidence] The source releases management using `removed` with
`destroy = false`. The target first declares the resource and then uses
`import` to adopt the object.

> [!WARNING]
> `moved` never performs a cross-state move. Review source release and target
> import as distinct state-boundary operations.

### Release Source Ownership

```hcl
removed {
   from = module.legacy.azurerm_linux_web_app.this

   lifecycle {
      destroy = false
   }
}
```

The source operation releases state ownership without requesting remote object
destruction. It does not prove target ownership.

### Adopt in the Target State

The target resource declaration must exist and match the import target.

```hcl
resource "azurerm_linux_web_app" "this" {
   name                = "api-prod"
   resource_group_name = "rg-platform-prod"
   location            = "eastus"
   service_plan_id     = var.service_plan_id
}

import {
   to = azurerm_linux_web_app.this
   id = "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rg-platform-prod/providers/Microsoft.Web/sites/api-prod"
}
```

[Terraform universal] The import identifier must be known during planning. The
identifier above is illustrative only.

1. Identify both roots, addresses, object ID, dependencies, and executor.
2. Create and validate target configuration without applying it.
3. Add target import and plan it; resolve unwanted update or replacement.
4. Review both plans for one source release and one target adoption.
5. Apply source `removed { destroy = false }` and verify state release.
6. Apply target import and verify target state owns the intended address.
7. Re-plan both roots for one owner and no unexplained action.

[Gap] Cross-state rollback, backups, recovery objectives, and backend-specific
recovery steps require organization policy. Do not present them as portable
Terraform behavior.

## Recognize Failure Modes

| Failure | Response |
| --- | --- |
| Rename plans create and destroy | Stop and correct the same-state mapping |
| Target import plans replacement | Reconcile target configuration before adoption |
| Source releases before target review | Pause and complete target review |
| Both roots manage the object | Stop and resolve one owner |
| Direct state editing seems necessary | Escalate the migration decision |

## Route Connections

Use the [published module upgrade route](../routes/published-module-upgrade.md)
for a consumed-module transition. Use the
[bespoke module authoring route](../routes/bespoke-module-authoring.md) when a
new module contract changes existing addresses. Include the migration record in
the [client handover route](../routes/client-handover.md) when operators inherit
the configuration.

## Related Core Guidance

Review [state and boundaries](state-and-boundaries.md),
[nested modules](nested-modules.md), and
[testing and change assurance](testing-and-change-assurance.md).