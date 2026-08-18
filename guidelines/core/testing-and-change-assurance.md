---
title: Terraform Testing and Change Assurance
description: Portable validation and evidence guidance for Terraform changes
---

## Build Assurance in Layers

Use checks that match the risk and scope of the change. Validate syntax and
configuration structure before assessing the proposed infrastructure change.
Then test module contracts and relevant integration behavior before authorizing
an apply.

[Terraform universal] `terraform validate` checks whether configuration is
internally valid. A plan compares the configuration with the current state and
provider information to propose infrastructure changes. Neither check proves an
organization approval or an operational recovery decision.

## Select Test Tiers

| Tier | Question | Mechanism |
| --- | --- | --- |
| Format and syntax | Is configuration valid? | `terraform fmt`, `terraform validate` |
| Contract validation | Do inputs meet declared rules? | Plan-mode `terraform test` |
| Mocked module test | Does module logic compose? | Mocked `terraform test` |
| Integration test | Does it work in a real environment? | Credentialed plan or test |
| Plan review | Is the root transition intended? | `terraform plan` |
| Apply verification | Did the operation reach intended state? | Profile-controlled apply |

[Training evidence] Local lessons distinguish validation, mocked unit-style,
and credentialed integration tiers. Keep their evidence separate so a mocked
pass is not reported as production integration evidence.

## Run Checks in Order

Use a backend-free sequence only for an isolated module fixture.

```bash
terraform fmt -check -recursive
terraform init -backend=false -input=false
terraform validate -no-color
terraform test -no-color
```

For a root that needs configured state, initialize normally and review its plan.

```bash
terraform fmt -check -recursive
terraform init
terraform validate -no-color
terraform plan -out=reviewed.tfplan
terraform show -no-color reviewed.tfplan
```

[Terraform universal] Saved plans can contain configuration, input values, and
sensitive values. Protect them and do not commit them to source control.

## Test Contracts With Plan Runs

[Terraform universal] Terraform discovers `.tftest.hcl` and `.tftest.json`
files. A test run defaults to `command = apply`; specify `command = plan` for
logic-only assertions.

```hcl
run "rejects_unknown_environment" {
	command = plan

	variables {
		environment = "sandbox"
	}

	expect_failures = [var.environment]
}
```

This original example expects a custom validation failure. Do not use
`expect_failures` to hide syntax, type, provider, or backend errors.

```hcl
run "plans_supported_environment" {
	command = plan

	variables {
		environment = "dev"
	}

	assert {
		condition     = output.name_prefix == "api-dev"
		error_message = "The output must include the environment prefix."
	}
}
```

## Test Module Logic With Mocks

[Terraform universal] Mock providers enable tests without real provider
operations. Their values are synthetic, so mocks test configuration logic and
output contracts, not cloud service behavior.

```hcl
mock_provider "azurerm" {}

run "applies_with_mocked_provider" {
	command = apply

	variables {
		environment = "dev"
	}
}
```

Mocks do not validate cloud permissions, quotas, policy, backend locks, remote
data sources, workflow identities, or provider API behavior.

## Use Integration Tests Deliberately

Integration tests use real provider behavior in an isolated non-production
environment. Decide scope, identity, cost and quota limits, cleanup behavior,
and evidence before execution.

[Gap] Required integration environments, identities, cleanup retention, and
approval model are organization policy. This page does not prescribe them.

## Distinguish Plan From Apply

Plans expose drift, replacement, imports, and address moves without applying.
Apply executes a plan or creates a new one and changes infrastructure. An
apply-style test can create real resources when it uses real providers.

| Operation | Establishes | Does not establish |
| --- | --- | --- |
| `terraform validate` | Internal configuration validity | Current infrastructure outcome |
| Plan-mode test | Assertions and validation failures | Provider API success |
| Mocked apply | Logic under synthetic behavior | Live provider behavior |
| `terraform plan` | Proposed observed-state transition | Successful execution |
| `terraform apply` | Execution result at that time | Future drift prevention |

Review source release and target import plans together for a cross-state
change. Plan address changes from representative prior state to verify `moved`
actions preserve identity. See [migration and refactoring](migration-and-refactoring.md).

## Clean Up Test Effects

[Terraform universal] `terraform test` uses separate in-memory test state, but
apply-style runs can create real resources. Terraform attempts cleanup and
reports resources it cannot remove.

Treat cleanup as a result. Record failures and retained-resource ownership.
Do not run broad `terraform destroy` against a shared root as generic cleanup.

[Gap] Failed-test retention, cleanup ownership, and recovery method require
organization policy.

## Run Lint and Security Checks

Run repository-provided lint and security tools after format, validation, tests,
and plan. Record each actual tool version, configuration, and finding
disposition. Do not invent a universal linter or scanner command.

## Retain Decision Evidence

Record the configuration revision, module versions, validation results, plan
review outcome, known limitations, and apply outcome. Associate the evidence
with the configuration and state boundary so an operator can understand what
was assessed.

The portable core does not prescribe a CI system, approval gate, saved-plan
artifact, identity mechanism, or retention period. Follow the selected
[execution profile](../index.md#execution-profiles) when those choices change
the assurance process.

## Test Change Classes

Include appropriate coverage for:

* Module inputs, outputs, validation, and compatibility expectations
* Provider requirements and aliased-provider wiring when used
* Address migrations, including `moved` blocks and upgrade paths
* Non-production integration behavior where cloud effects matter
* Destructive, replacement, or boundary-changing operations

## Route Connections

Apply assurance while following the
[bespoke module authoring route](../routes/bespoke-module-authoring.md), the
[published module upgrade route](../routes/published-module-upgrade.md), or
the [client handover route](../routes/client-handover.md).

## Related Core Guidance

Use [migration and refactoring](migration-and-refactoring.md) for address
changes and [handover evidence packs](handover-evidence-pack.md) for the
records operators need.