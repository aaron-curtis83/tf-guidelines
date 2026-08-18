---
title: Terraform Modules and Versioning
description: Portable guidance for module contracts, compatibility, and controlled upgrades
---

## Define a Module Contract

Define the module's purpose, inputs, outputs, provider requirements, Terraform
version requirements, and documented behavior before consumers depend on it.
Treat names, types, defaults, validation rules, outputs, and behavior that a
consumer uses as compatibility-sensitive contract elements.

[Terraform universal] A module call selects a source and may select a version.
The calling configuration supplies module inputs and consumes declared module
outputs. Changes to those interfaces can require a consumer configuration
change even when the underlying resources do not change.

## Version With Consumer Impact

Publish version changes that allow consumers to identify compatible upgrades.
Describe breaking input, output, provider, Terraform-version, and behavior
changes in the release notes. Test the supported upgrade path rather than
assuming that a successful module test proves consumer compatibility.

The organization has not selected a module registry or release policy. Record
the source, version selection, support expectation, and upgrade evidence for
each module until that policy is approved.

## Upgrade a Consumed Module

Before changing a consumed module version:

1. Read the target version's compatibility and migration notes.
2. Compare the current and target input, output, provider, and Terraform
   requirements.
3. Run the relevant validation and plan using the consumer configuration.
4. Assess address or state effects and preserve migration instructions.
5. Capture the selected delivery evidence.

## Route Connections

Use the [bespoke module authoring route](../routes/bespoke-module-authoring.md)
to establish a new module contract. Use the
[published module upgrade route](../routes/published-module-upgrade.md) for a
consumed-module change. Include contract and version evidence in the
[client handover route](../routes/client-handover.md).

## Related Core Guidance

See [nested modules](nested-modules.md) for the limited packaged-composition
exception, [testing and change assurance](testing-and-change-assurance.md) for
compatibility evidence, and [migration and refactoring](migration-and-refactoring.md)
for address changes.

## Contract Inventory and Layout

| Contract element | Consumer dependency | Label |
| --- | --- | --- |
| Input name, type, default, validation | Caller configuration | `[Terraform universal]` |
| Provider and Terraform requirements | Dependency selection | `[Terraform universal]` |
| Output name, type, and meaning | Downstream configuration | `[Terraform universal]` |
| Release and migration notes | Controlled upgrade | `[Practice recommendation]` |

| File | Owns | Must not own | Label |
| --- | --- | --- | --- |
| `main.tf` | Resources and composition | Root backend configuration | `[Practice recommendation]` |
| `variables.tf` | Public inputs and validation | Derived naming rules | `[Practice recommendation]` |
| `locals.tf` | Internal computations | Caller-selected settings | `[Practice recommendation]` |
| `outputs.tf` | Supported consumer results | Incidental internals | `[Practice recommendation]` |
| `terraform.tf` | Terraform and provider requirements | Credentials | `[Terraform universal]` |

[Terraform universal] Reusable child modules declare provider requirements;
provider configurations belong in the root module.

## Typed Contract

```hcl
variable "service" {
   type = object({
      name               = string
      enable_diagnostics = optional(bool, true)
   })

   validation {
      condition     = length(trimspace(var.service.name)) > 0
      error_message = "service.name must not be empty."
   }
}

locals {
   resource_name = var.service.name
}
```

[Terraform universal] The `optional` attribute supplies its default when the
caller omits that attribute.

[Practice recommendation] Keep `resource_name` internal unless callers must
intentionally override the naming contract.

```hcl
output "service_name" {
   description = "Stable name for the managed service"
   value       = local.resource_name
}
```

| Output change | Release impact | Label |
| --- | --- | --- |
| Add optional output | Usually non-breaking | `[Practice recommendation]` |
| Rename or remove output | Breaking | `[Practice recommendation]` |
| Change type or meaning | Breaking unless proven otherwise | `[Practice recommendation]` |
| Mark output sensitive | Assess display and consumers | `[Terraform universal]` |

## Provider and Source Selection

```hcl
terraform {
   required_version = ">= 1.7.0"

   required_providers {
      azurerm = {
         source  = "hashicorp/azurerm"
         version = ">= 4.0, < 5.0"
      }
   }
}
```

[Terraform universal] A provider constraint chooses eligible dependency
versions. It does not guarantee module-behavior compatibility.

```hcl
module "observability_from_git" {
   source = "git::https://example.invalid/platform/observability.git?ref=v2.3.1"
}

module "observability_from_registry" {
   source  = "example/observability/azurerm"
   version = "2.3.1"
}
```

[Terraform universal] `?ref=v2.3.1` is part of a Git source address, not a
registry `version` constraint.

[Practice recommendation] Use a reproducible revision and record its support
expectation. [Gap] DR-01 does not prescribe tags, commits, or a registry model.

## SemVer and Upgrade Checks

| Change | Suggested impact | Evidence | Label |
| --- | --- | --- | --- |
| Fix with unchanged contract | Patch | Focused plan | `[Practice recommendation]` |
| Optional capability | Minor | Existing plan remains stable | `[Practice recommendation]` |
| Removed or incompatible interface | Major | Migration and consumer plan | `[Practice recommendation]` |
| Terraform or provider floor increase | Assess as breaking | Prerequisites tested | `[Practice recommendation]` |
| Address refactor | Separate state impact | `moved` evidence | `[Terraform universal]` |

1. Record the current source and version.
2. Read target release and migration notes.
3. Compare inputs, outputs, provider, Terraform, and behavior contracts.
4. Run `terraform init -upgrade` when dependency selection changes.
5. Run `terraform validate` and a consumer-scoped `terraform plan`.
6. Inspect addresses, replacements, and output changes.
7. Record test results and remaining assumptions.

[Terraform universal] `terraform init -upgrade` updates installed selections
that satisfy configured constraints.

[Practice recommendation] Test the supported upgrade path in a consumer,
because a module-only test cannot prove all consumer compatibility.