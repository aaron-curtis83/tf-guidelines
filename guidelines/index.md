---
title: Terraform Practice Guidelines Library
description: Route-first navigation for Terraform module authoring, upgrades, and client handover
---

## Start With Your Task

Choose the route that matches the work you need to complete. Each route leads
to portable Terraform guidance, then to the selected execution profile and
platform overlay when delivery behavior differs.

* [Author a bespoke module](routes/bespoke-module-authoring.md)
* [Upgrade a published module](routes/published-module-upgrade.md)
* [Prepare a client handover](routes/client-handover.md)

### Entry Criteria

Start here when you can name the configuration or module affected by the
work. You do not need a final backend, registry, approval, or retention
decision before choosing a route. Record unknowns as assumptions or gaps
rather than inferring an organization standard.

| Reader | Start when you know | First outcome to record |
| --- | --- | --- |
| Module author | The capability, intended consumers, and expected lifecycle | Contract and ownership questions |
| Module consumer | The current source, version, and configuration roots that consume it | Compatibility and state-impact assessment |
| Client operator | The configuration root or state boundary being transferred | Evidence-pack owner and missing records |

> [!IMPORTANT]
> Select one apply authority for each state boundary. A route does not itself
> authorize an apply, name a backend, or choose a platform control.

### Fast Task Triage

Use the first matching condition to select the route. If more than one
condition applies, complete the route that controls the current risk first.

| Task signal | Route | Why it starts there |
| --- | --- | --- |
| New capability, private module, or new reusable contract | [Bespoke module authoring](routes/bespoke-module-authoring.md) | The contract and state design are still open |
| Changed `source` or version for an existing module call | [Published module upgrade](routes/published-module-upgrade.md) | Compatibility and address effects need review before delivery |
| Operational responsibility moves to a client team | [Client handover](routes/client-handover.md) | Operators need evidence, access context, and escalation paths |
| State must move between roots or addresses | [Migration and refactoring](core/migration-and-refactoring.md) | Migration classification precedes any route-specific delivery work |
| The apply model is unknown | [Execution-profile selector](execution-profiles/select-an-execution-profile.md) | The plan-to-apply relationship must be explicit |

### Minimum Information to Gather

Collect the following facts before proceeding beyond the first route step.
They are a navigation aid, not a request for a specific policy choice.

* Repository and configuration-root path
* Affected module source and version, when a module is consumed
* Intended state boundary and the team accountable for its operation
* Change type: contract, dependency, state address, infrastructure, or handover
* Selected or proposed apply authority
* Known dependency, identity, recovery, retention, or ownership uncertainty

An incomplete answer is useful when it is recorded as `[Assumption]` or
`[Gap]` and assigned to an owner in the
[known gaps and assumptions](reference/known-gaps-and-assumptions.md) register.

## Navigation Model

Follow this order when a route links to shared guidance:

```text
Route page -> Portable Terraform core -> Selected execution profile -> Platform overlay
```

The sequence separates four kinds of decision. Routes frame the task. Core
pages describe portable Terraform behavior. Profiles define how a selected
apply model links review to execution. Overlays describe controls owned by a
named platform. Do not use an overlay to establish a portable Terraform rule.

### Navigation Decision Matrix

| Question | Use this page | Record before continuing |
| --- | --- | --- |
| Is the work a module, root, state, or handover concern? | [Decision pathways](core/decision-pathways.md) | Scope and accountable owner |
| Which resources must share lifecycle and apply authority? | [State and boundaries](core/state-and-boundaries.md) | Proposed root and state boundary |
| Is the public module contract compatible? | [Modules and versioning](core/modules-and-versioning.md) | Consumer impact and version decision |
| Does an address or state ownership change? | [Migration and refactoring](core/migration-and-refactoring.md) | Same-state or cross-state classification |
| What assurance is proportionate to the change? | [Testing and change assurance](core/testing-and-change-assurance.md) | Checks, evidence, and unresolved risk |
| How will review connect to an apply? | [Execution-profile selector](execution-profiles/select-an-execution-profile.md) | Chosen profile and apply authority |
| What must transfer to operations? | [Handover evidence pack](core/handover-evidence-pack.md) | Evidence owner, freshness, and escalation path |

### Route Completion Records

Each route ends in a measurable record. Store the record with the work item,
change request, or delivery evidence location selected by the delivery team.
The library does not prescribe that storage location.

| Route | Completion record | Minimum measurable fields |
| --- | --- | --- |
| Bespoke module authoring | Module delivery decision | Contract owner, state boundary, profile, assurance result |
| Published module upgrade | Upgrade decision | Current and target versions, compatibility result, plan review result |
| Client handover | Handover index | Root, state or workspace owner, evidence locations, open gaps |

### Worked Navigation Record

The following original record shows how a delivery lead can link a task to
evidence without claiming a policy choice.

```text
task: Upgrade payments configuration from module version 2.3.1 to 3.0.0
route: published-module-upgrade
state_boundary: payments-production
apply_authority: runner-managed-reviewed-plan
evidence: compatibility note, reviewed plan record, apply outcome
open_item: [Gap] retention duration pending records owner
```

The route remains valid if the state backend or artifact retention setting is
not yet approved. The open item remains visible instead of being silently
converted into a requirement.

### Portable Core Guidance

The portable core covers [delivery lifecycle](core/delivery-lifecycle.md),
[decision pathways](core/decision-pathways.md),
[state and boundaries](core/state-and-boundaries.md),
[modules and versioning](core/modules-and-versioning.md),
[nested modules](core/nested-modules.md),
[testing and change assurance](core/testing-and-change-assurance.md),
[migration and refactoring](core/migration-and-refactoring.md), and
[handover evidence packs](core/handover-evidence-pack.md).

Use core pages to interpret portable behavior. For example,
`terraform plan -out` and `terraform apply PLANFILE` are Terraform CLI
semantics, while artifact upload, retention, and approval controls belong to
the selected platform. The
[evidence catalog](reference/evidence-catalog.md) records this distinction.

### Execution Profiles

Select one apply authority for each configuration with the
[execution-profile selector](execution-profiles/select-an-execution-profile.md).
The library distinguishes runner-managed reviewed-plan execution from HCP
Terraform VCS workspace execution.

### Profile Selection Guardrails

| Selected model | Review-to-apply relationship | Do not assume |
| --- | --- | --- |
| Runner-managed reviewed plan | A runner applies the specific reviewed saved plan it selected and verified | That every CI system supplies the same artifact, approval, or retention controls |
| HCP Terraform VCS workspace | HCP Terraform creates a normal run from the merged revision | That a pull-request speculative plan is later applied |

[Execution profile] Do not combine these models for one state boundary. A
GitHub Actions or Azure DevOps runner can host the runner-managed model, but
that does not make the root an HCP Terraform VCS workspace.

### Platform Overlays

Platform overlays describe controls owned by [GitHub Actions](platform-overlays/github-actions.md),
[Azure DevOps](platform-overlays/azure-devops.md), and
[HCP Terraform](platform-overlays/hcp-terraform.md). They do not replace
portable Terraform guidance.

### Platform Selection Notes

* [GitHub Actions] Use the GitHub overlay only for controls and observed
	workflow patterns owned by GitHub Actions or its reusable-workflow contract.
* [Azure DevOps] Use the Azure DevOps overlay only for Pipelines, service
	connections, Pipeline Artifacts, branch policies, or Environment controls.
* [HCP Terraform] Use the HCP Terraform overlay for workspace, remote-run,
	policy-set, and VCS-workflow behavior.

If a control cannot be confirmed from the selected platform's evidence, record
it as a gap. Do not transfer mechanics from one overlay to another.

## Evidence Labels

Use the following labels to identify the source and scope of guidance.

| Label | Meaning |
| --- | --- |
| `[Terraform universal]` | Portable Terraform language or HashiCorp behavior |
| `[Execution profile]` | Required behavior of the selected runner-managed or HCP Terraform model |
| `[GitHub Actions]` | GitHub-owned control or local implementation example |
| `[Azure DevOps]` | Azure DevOps-owned control or local implementation example |
| `[HCP Terraform]` | HCP Terraform workspace, run, or policy behavior |
| `[Gold-standard example]` | Current internal implementation, not a universal mandate |
| `[Practice recommendation]` | Organization guidance derived from evidence that needs governance approval |
| `[Assumption]` | Input awaiting independent validation |
| `[Gap]` | Topic without sufficient proof for a normative rule |

### How to Read a Label

Labels qualify a claim rather than rank it. Read the label before reusing a
statement in a design, template, or review comment.

| If a statement has | You can treat it as | You must not treat it as |
| --- | --- | --- |
| `[Terraform universal]` | Portable Terraform or HashiCorp-documented behavior | A platform approval or organization operating policy |
| `[Execution profile]` | A rule that follows from the selected apply model | A rule for a different profile |
| A platform label | Behavior or control in that named platform boundary | A control automatically present in another platform |
| `[Gold-standard example]` | Evidence of a current internal implementation | Proof that the pattern is mandated or fully configured |
| `[Practice recommendation]` | Guidance requiring governance approval | An approved mandate |
| `[Assumption]` or `[Gap]` | An item requiring validation or ownership | Permission to invent a resolution |

### Source Interpretation

[Terraform universal] HashiCorp documentation supports claims about Terraform
language, CLI, state, provider, test, import, and HCP Terraform behavior.
Local lesson sources illustrate the learning sequence and local
examples. [Gold-standard example] Internal asset research records observed
implementations. The
[evidence catalog](reference/evidence-catalog.md) links each source category
to the pages where it can be used.

When a source is not listed in the catalog, add an evidence record before
using it to support a new guideline claim. Cite the source path or HashiCorp
link, describe the limited claim it supports, and identify its label.

## Unresolved Policy

Organization decisions for state backends, module registries, HCP Terraform
edition, retention, recovery, and handover ownership remain unresolved. Treat
these as escalation points until the [reference records](reference/known-gaps-and-assumptions.md)
are resolved.

### Escalation Record

Use this original, short record when a route cannot continue safely without an
owner decision.

```text
label: [Gap]
topic: State recovery procedure for payments-production
decision_needed: Define recovery owner and evidence location
affected_route: client-handover
interim_action: Record current contacts and do not state a recovery baseline
owner: Operations and platform owner
```

Escalation identifies the missing decision without blocking independent work
such as module compatibility review or evidence inventory.

## Verification Before You Leave

Confirm the following before handing work to a route, profile, or overlay:

* The selected route matches the current task, not a future follow-on task
* The configuration root and state boundary are identified or explicitly open
* The reader can reach the required core page, profile, and applicable overlay
* Every open policy decision is recorded as `[Assumption]` or `[Gap]`
* The route's completion record has a named evidence location and owner

Return to the route page when these checks pass. Use the
[glossary](reference/glossary.md) when terminology is unclear and the
[known gaps and assumptions](reference/known-gaps-and-assumptions.md) register
when a required decision lacks sufficient evidence.