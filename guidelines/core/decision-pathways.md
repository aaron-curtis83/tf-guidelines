---
title: Terraform Decision Pathways
description: Decision and escalation guidance for Terraform configuration, state, module, and delivery choices
---

## Start With the Decision

Make the decision at the boundary where it changes ownership, lifecycle,
compatibility, or state. Do not select a platform control to resolve a
Terraform design question.

## Decision Questions

Use these questions in order:

1. What capability is being managed, and who owns its lifecycle?
2. Does it need an independently deployable root and state boundary?
3. Is the capability a reusable module contract or a configuration-specific
   composition?
4. Can consumers upgrade it independently, and what compatibility promise is
   required?
5. Does the change alter a Terraform address, provider configuration, or
   existing managed object?
6. Which execution authority owns review and apply behavior?
7. What evidence must an operator receive at handover?

## Escalate Unknown Policy

The library does not establish organization policy for state backend, module
registry, retention, recovery, HCP Terraform edition, or handover ownership.
Record the decision owner and the assumption, then escalate it through the
relevant governance path. Do not promote a project convention into portable
guidance without evidence and approval.

## Route Connections

* Use the [bespoke module authoring route](../routes/bespoke-module-authoring.md)
  when a new contract or composition is required.
* Use the [published module upgrade route](../routes/published-module-upgrade.md)
  when the decision concerns a consumed module version.
* Use the [client handover route](../routes/client-handover.md) when ownership
  or operator evidence is changing.

## Related Core Guidance

Apply the resulting decision through [state and boundaries](state-and-boundaries.md),
[modules and versioning](modules-and-versioning.md), and the
[delivery lifecycle](delivery-lifecycle.md).

## Boundary Matrix

[Practice recommendation] Decide a root boundary by lifecycle, ownership,
blast radius, dependency direction, and environment isolation together.

| Test | Inspect | Risk if ignored | Label |
| --- | --- | --- | --- |
| Lifecycle | Deploy and retirement cadence | Coupled releases | `[Practice recommendation]` |
| Ownership | Accountable operating team | Unclear apply authority | `[Practice recommendation]` |
| Blast radius | Objects affected by failed apply | Wider recovery event | `[Practice recommendation]` |
| Dependency | Upstream outputs and consumers | Circular coupling | `[Practice recommendation]` |
| Environment | Required isolation | Cross-environment impact | `[Practice recommendation]` |

[Terraform universal] State binds remote objects to Terraform resource
instances. Address changes can therefore alter how an existing object is managed.

[Gap] Backend selection, locking implementation, and recovery procedure remain
DR-01 items in the [known gaps register](../reference/known-gaps-and-assumptions.md).

## Inputs and Locals

Use an input for caller intent. Use a local for a repeatable computation.

| Value | Prefer | Reason | Label |
| --- | --- | --- | --- |
| Deployment environment | Typed input | Caller chooses it | `[Terraform universal]` |
| Derived resource name | Local | Avoids duplicate choices | `[Practice recommendation]` |
| Parent-derived child value | Child argument | Locals do not cross module boundaries | `[Terraform universal]` |
| CLI-redacted value | Sensitive input | Redaction is not state secrecy | `[Terraform universal]` |

[Terraform universal] A `validation` block evaluates a custom input condition.
`sensitive = true` redacts CLI output but does not by itself prevent state storage.

```hcl
variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "prod"], var.environment)
    error_message = "environment must be dev or prod."
  }
}

locals {
  name_prefix = "api-${var.environment}"
}
```

[Practice recommendation] Verify this original example with `terraform fmt
-check` and `terraform validate` in a configuration that declares the variable.

## Migration and Source Decisions

| Change | Required review | Label |
| --- | --- | --- |
| Rename resource or module | Add and test `moved` | `[Terraform universal]` |
| Change `count` or `for_each` key | Inventory old and new addresses | `[Terraform universal]` |
| Move object between states | Use cross-state procedure | `[Practice recommendation]` |
| Remove module input | Release as breaking | `[Practice recommendation]` |
| Change output meaning | Test downstream callers | `[Practice recommendation]` |

[Terraform universal] A `moved` block maps addresses during planning within one
state boundary. It does not transfer an object between independent states.

| Consumer need | Source form | Version selector | Label |
| --- | --- | --- | --- |
| Local development | Relative path | Working tree | `[Terraform universal]` |
| Git-hosted module | Git source | Optional `?ref=` | `[Terraform universal]` |
| Registry module | Registry address | `version` argument | `[Terraform universal]` |
| Supported upgrade | Published source | Tested revision | `[Practice recommendation]` |

[Terraform universal] The `version` argument applies to registry sources. Git
source selection uses its source address with an optional `?ref=` query.

[Gap] DR-01 does not require a registry, tag policy, or publication gate.

## Execution and Escalation

[Practice recommendation] Select one execution profile for each state boundary.

| Condition | Select | Do not infer | Label |
| --- | --- | --- | --- |
| Delivery applies selected saved plan | Runner-managed reviewed plan | Reviewer approval | `[Execution profile]` |
| VCS workspace runs after merge | HCP Terraform VCS workspace | Saved-plan provenance | `[Execution profile]` |
| Identity or approval is unknown | Escalate to owner | Vendor default | `[Gap]` |

[Terraform universal] `terraform apply PLANFILE` executes the saved plan without
an interactive plan approval prompt. Terraform does not identify reviewers.

```text
capability: shared-private-endpoint
question: Does it require an independent state boundary?
known_facts: Platform and application teams consume it
assumption: One owner can approve lifecycle changes
required_decision: State owner and apply authority
interim_boundary: Do not share state without named ownership
```

[Assumption] Validate the named owner before treating the interim boundary as a
team rule.

## Review Checklist

* Is there one named lifecycle owner?
* Does the root and state match deployable blast radius?
* Are caller choices typed inputs and derivations locals?
* Are source syntax and compatibility impact recorded?
* Does the selected executor match the state boundary?
* Is every unresolved choice a labeled gap or assumption?

[Practice recommendation] Add these answers to the change and handover record.