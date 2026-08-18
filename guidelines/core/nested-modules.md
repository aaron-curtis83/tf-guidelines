---
title: Nested Modules: Exceptional Packaged Composition
description: Bounded guidance for a reusable Terraform parent module that packages internal child modules
---

## Default to Flat Root Composition

Prefer a root configuration that composes peer modules when components need
independent lifecycle, ownership, versioning, deployment, replacement, or
consumption. This keeps each module contract visible to the root caller and
allows each capability to evolve independently.

## Permit Packaged Composition Only for a Cohesive Capability

Use a reusable parent module with internal child modules only when one team
owns and releases a cohesive, opinionated capability as one package. The parent
and children must share a lifecycle, version, release, and support boundary.
Keep the tree shallow, normally one parent-to-child level, and keep bundled
children under `modules/` with relative sources.

[Terraform universal] A module can call another module using a module block.
A relative source identifies a child stored with the parent module. Terraform
evaluates the child through the calling module, while the root caller remains
responsible for provider configuration.

## Keep the Public Contract at the Parent Boundary

Treat the parent inputs, outputs, required Terraform version, provider
requirements, and documented behavior as the public contract. Keep child
labels, paths, and child-only outputs internal unless there is independent
evidence and approval to support them as public interfaces. Release bundled
children with the parent rather than treating their paths as independently
versioned products.

## Configure Providers at the Root Caller

[Terraform universal] Every module declares the provider requirements it uses.
Provider configurations belong in the root caller. Pass a provider alias
explicitly to any child that needs it, and declare `configuration_aliases` in
each module that accepts an alias. Do not rely on an internal child implicitly
discovering a caller's aliased configuration.

## Test, Release, and Migrate as One Package

Test child contracts, parent wiring, and non-production integration behavior
when cloud behavior matters. Release the parent and its bundled children as one
package. When a child path, parent path, or caller address changes, treat it as
a state migration: test the supported upgrade paths and publish the required
`moved` blocks with the release.

## Evidence Boundary

The local lessons cover module structure, input and output contracts, and
versioning. They do not directly teach reusable parent-child composition.
The `[Terraform universal]` statements above identify portable Terraform
language and provider behavior. The flat-root default and the conditions for
this exception are practice guidance that require organization approval before
they become a mandatory policy. Independent consumption or release of a bundled
child remains an unresolved policy decision.

## Route Connections

Use the [bespoke module authoring route](../routes/bespoke-module-authoring.md)
to decide whether the capability qualifies for this exception. Use the
[published module upgrade route](../routes/published-module-upgrade.md) to
assess a packaged release change. Include the composition and migration record
in the [client handover route](../routes/client-handover.md).

## Related Core Guidance

Continue with [modules and versioning](modules-and-versioning.md),
[migration and refactoring](migration-and-refactoring.md), and
[testing and change assurance](testing-and-change-assurance.md).

[Practice recommendation] Flat-root composition is the default. Compose peer
modules from the root when components need independent ownership, lifecycle,
release, replacement, or consumption.

[Assumption] The exception is one tightly bounded cohesive package owned,
released, supported, and retired by one team.

[Gap] DR-02 has not approved independent consumption or release of bundled
children. Do not promise either behavior.

| Question | Required answer for exception | Label |
| --- | --- | --- |
| Capability | One opinionated product capability | `[Practice recommendation]` |
| Ownership | One accountable team | `[Practice recommendation]` |
| Lifecycle | Parent and children deploy and retire together | `[Practice recommendation]` |
| Release | One version and support statement | `[Assumption]` |
| Consumer contract | Root callers need only parent interface | `[Practice recommendation]` |

[Practice recommendation] A child needed by another root is evidence against
bundling because that reuse creates an independent consumer contract.

[Terraform universal] A relative source beginning `./` resolves from the
calling module directory.

```text
private-service/
	main.tf
	variables.tf
	outputs.tf
	terraform.tf
	modules/
		dns/
			main.tf
			variables.tf
			outputs.tf
			terraform.tf
```

```hcl
module "dns" {
	source = "./modules/dns"

	zone_name    = var.zone_name
	service_fqdn = local.service_fqdn
}
```

[Practice recommendation] The tree identifies a release bundle, not a promise
that `modules/dns` is independently consumable.

| Interface | Visibility | Label |
| --- | --- | --- |
| Parent input `zone_name` | Public | `[Practice recommendation]` |
| Parent output `service_endpoint` | Public | `[Practice recommendation]` |
| Child source path | Internal | `[Practice recommendation]` |
| Child output `record_id` | Internal | `[Practice recommendation]` |

```hcl
output "service_endpoint" {
	description = "Fully qualified endpoint for the packaged service"
	value       = module.dns.service_endpoint
}
```

[Terraform universal] A child output is available to its direct parent. It is
not automatically exposed to the root caller.

## Provider Alias Forwarding

[Terraform universal] A module accepting an alias declares it in
`required_providers.configuration_aliases`; the root configures the instance
and module calls map it explicitly.

```hcl
terraform {
	required_providers {
		azurerm = {
			source                = "hashicorp/azurerm"
			configuration_aliases = [azurerm.connectivity]
		}
	}
}
```

```hcl
module "dns" {
	source = "./modules/dns"

	providers = {
		azurerm.connectivity = azurerm.connectivity
	}
}
```

[Terraform universal] The declaration accepts an alias; it does not configure
it. The root remains responsible for provider configuration.

[Practice recommendation] Verify source resolution with `terraform init
-backend=false`, then run `terraform validate` and a plan fixture with the root alias.

## Bundle Release and Migration

| Check | Proves | Label |
| --- | --- | --- |
| Format and validation | HCL and requirements shape | `[Terraform universal]` |
| Parent fixture | Inputs, outputs, and alias wiring | `[Practice recommendation]` |
| Integration fixture | Selected provider behavior | `[Practice recommendation]` |
| Consumer upgrade | Package compatibility | `[Assumption]` |

[Practice recommendation] Release the parent and bundled children as one
source revision and one compatibility statement.

[Terraform universal] Module paths contribute to resource addresses. Moving a
child call or renaming a parent can therefore change addresses.

```hcl
moved {
	from = module.private_service.module.dns
	to   = module.private_service.module.private_dns
}
```

[Terraform universal] The original example maps addresses within one state. It
is not a cross-state transfer.

1. Identify the installed parent source and version.
2. Compare parent inputs, outputs, Terraform, and provider requirements.
3. Confirm child paths remain internal implementation details.
4. Run `terraform init -upgrade` when source selection changes.
5. Run `terraform validate` and a consumer-scoped plan.
6. Inspect address moves, replacements, and alias errors.
7. Test migration against representative state.
8. Record bundle version, test result, and DR-02 boundary.

| Failure | Correction | Label |
| --- | --- | --- |
| Child used directly by another root | Publish a contract or stop direct use | `[Assumption]` |
| Alias not declared by accepting child | Add `configuration_aliases` | `[Terraform universal]` |
| Provider configured in child | Configure it in root | `[Terraform universal]` |
| Independent child release promised | Escalate DR-02 | `[Gap]` |
| Package split without address plan | Classify and test migration | `[Practice recommendation]` |

[Practice recommendation] Record package owner, parent source and version,
child inventory, aliases, tests, migration history, and DR-02 at handover.