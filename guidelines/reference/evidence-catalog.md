---
title: Evidence Catalog
description: Source categories and claim boundaries for the Terraform Practice Guidelines
---

## Label Model

Use one or more labels at the start of a claim. Labels identify the evidence
boundary, not the importance of the claim.

| Label                       | Meaning                                                                    |
|-----------------------------|----------------------------------------------------------------------------|
| `[Terraform universal]`     | Portable Terraform language or HashiCorp behavior                          |
| `[Execution profile]`       | Required behavior of the selected runner-managed or HCP Terraform model    |
| `[GitHub Actions]`          | GitHub-owned control or local implementation example                       |
| `[Azure DevOps]`            | Azure DevOps-owned control or local implementation example                 |
| `[HCP Terraform]`           | HCP Terraform workspace, run, or policy behavior                           |
| `[Gold-standard example]`   | Current internal implementation, not a universal mandate                   |
| `[Practice recommendation]` | Organization guidance derived from evidence that needs governance approval |
| `[Assumption]`              | Input awaiting independent validation                                      |
| `[Gap]`                     | Topic without sufficient proof for a normative rule                        |

### Applying a Label

Put the applicable label at the beginning of the claim it qualifies. Use more
than one label only when each source boundary is needed to understand the
claim. A label does not make a statement more or less important; it tells the
reader how far the statement can travel.

| Claim type | Appropriate label | Example interpretation |
| --- | --- | --- |
| Terraform CLI or language behavior | `[Terraform universal]` | A saved plan is a Terraform artifact, regardless of where it runs |
| Behavior required by one apply model | `[Execution profile]` | A runner-managed flow applies the selected saved plan |
| Behavior configured in one platform | A named platform label | Environment checks belong to the selected platform, not Terraform itself |
| Observed internal implementation | `[Gold-standard example]` | A source repository demonstrates a current pattern only |
| Proposed organizational direction | `[Practice recommendation]` | The proposal awaits governance approval |
| Missing or unverified input | `[Assumption]` or `[Gap]` | The reader must obtain a decision before claiming a baseline |

### Claim Construction Pattern

Write a claim in four short parts when the source boundary might be unclear:

1. State the behavior or observation.
2. Apply the evidence label.
3. Cite the internal source path or HashiCorp link.
4. State the boundary that the evidence does not establish.

For example: `[Gold-standard example] Current GitHub callers select a
pull-request run by head SHA before applying its plan artifact. Source:
`.copilot-tracking/research/subagents/2026-08-18/github-delivery-mechanics-gap-analysis.md`.
This does not prove reviewer protection, artifact attestation, or a universal
GitHub baseline.`

## Claim Sources

| Claim area                     | Label or labels                                      | Source category                                                     | Guideline locations                                                                 |
|--------------------------------|------------------------------------------------------|---------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| State, module, test, and move behavior | `[Terraform universal]`                     | HashiCorp Terraform language and CLI documentation                 | [State and boundaries](../core/state-and-boundaries.md), [modules and versioning](../core/modules-and-versioning.md), [migration and refactoring](../core/migration-and-refactoring.md), [testing and change assurance](../core/testing-and-change-assurance.md) |
| Parent and child module behavior | `[Terraform universal]` `[Practice recommendation]` | HashiCorp module and provider documentation; Lessons 4, 5, and 7   | [Nested modules](../core/nested-modules.md)                                        |
| Runner-owned saved-plan lifecycle | `[Execution profile]`                       | Terraform saved-plan behavior and Lesson 6 delivery evidence       | [Runner-managed reviewed plan](../execution-profiles/runner-managed-reviewed-plan.md) |
| GitHub runner contract          | `[GitHub Actions]` `[Gold-standard example]` | Internal GitHub templates and reusable workflow source              | [GitHub Actions overlay](../platform-overlays/github-actions.md)                   |
| Azure DevOps delivery controls  | `[Azure DevOps]`                              | Azure DevOps Pipeline documentation and Lesson 6 delivery evidence | [Azure DevOps overlay](../platform-overlays/azure-devops.md)                       |
| HCP workspace lifecycle         | `[HCP Terraform]` `[Execution profile]`      | HCP Terraform workspace and VCS workflow documentation             | [HCP Terraform VCS workspace](../execution-profiles/hcp-terraform-vcs-workspace.md), [HCP Terraform overlay](../platform-overlays/hcp-terraform.md) |
| Organizational policy decisions | `[Practice recommendation]` `[Assumption]` `[Gap]` | No approved organization policy available in local evidence         | [Known gaps and assumptions](known-gaps-and-assumptions.md)                        |

### Source Register

The following sources can support the listed claim categories. Do not treat a
source as authority for categories outside its stated scope.

| Source | Classification | Supports | Does not establish |
| --- | --- | --- | --- |
| [Input variables](https://developer.hashicorp.com/terraform/language/values/variables) | HashiCorp authoritative | Variable type, validation, sensitive, and ephemeral semantics | An organization secret-storage policy |
| [Providers within modules](https://developer.hashicorp.com/terraform/language/modules/develop/providers) | HashiCorp authoritative | Root provider configuration, child requirements, and aliases | A provider-selection or identity baseline |
| [Purpose of Terraform state](https://developer.hashicorp.com/terraform/language/state/purpose) | HashiCorp authoritative | State mapping and collaboration rationale | A chosen backend, recovery objective, or retention period |
| [Backend configuration](https://developer.hashicorp.com/terraform/language/backend) | HashiCorp authoritative | Backend configuration constraints and initialization behavior | An approved backend product or access model |
| [Terraform tests](https://developer.hashicorp.com/terraform/language/tests) | HashiCorp authoritative | Test files, run blocks, plan or apply behavior, and mocks | Required test tier or cloud-account policy |
| [Refactor modules](https://developer.hashicorp.com/terraform/language/modules/develop/refactoring) | HashiCorp authoritative | `moved` behavior and historical move compatibility | A cross-state migration approval process |
| [Import block](https://developer.hashicorp.com/terraform/language/block/import) | HashiCorp authoritative | Import declaration requirements and provider-specific IDs | Ownership approval for existing infrastructure |
| `terraform-training-lesson-04/lessons/section-02-module-structure.md` | Local lesson | Module layout and root-versus-module teaching examples | Publication, support, or registry policy |
| `terraform-training-lesson-06/lessons/section-04-azure-devops.md` | Local lesson | Azure DevOps delivery pattern concepts | A verified live Pipeline, Environment, or service-connection contract |
| `terraform-training-lesson-08/lessons/section-05-moved-blocks.md` | Local lesson | Same-state migration learning sequence | A full rollback or cross-state operating procedure |
| `.copilot-tracking/research/subagents/2026-08-18/gold-standard-assets-research.md` | Gold-standard research | Current internal workflow and test patterns | Mandatory enterprise configuration |
| `.copilot-tracking/research/subagents/2026-08-18/github-delivery-mechanics-gap-analysis.md` | Gold-standard research | Observed GitHub plan provenance mechanics | GitHub platform defaults or approved policy |

### Evidence Selection Matrix

| Reader question | Select evidence | Follow-up action |
| --- | --- | --- |
| What does Terraform itself do? | HashiCorp authoritative documentation | Label the portable claim `[Terraform universal]` |
| How do the training lessons introduce this practice? | Relevant local lesson path | Label as local lesson context, not a universal policy |
| How does an internal implementation currently behave? | Gold-standard research record | Label `[Gold-standard example]` and state limitations |
| What platform setting must exist? | Named platform evidence and configuration owner | Use platform label or record a gap |
| What is our organization required to do? | Approved policy or decision record | Do not claim a requirement until approval exists |

### Original Evidence Record

Use this original record format to attach a claim to its source without
inventing a policy:

```text
claim: A child module declares an accepted provider alias; the root passes it.
label: [Terraform universal]
source: https://developer.hashicorp.com/terraform/language/modules/develop/providers
used_by: guidelines/core/nested-modules.md
boundary: Does not select AzureRM, identity claims, or root provider settings.
reviewed: 2026-08-18
```

The `boundary` field is mandatory when a reader could otherwise infer an
organizational control from a portable Terraform statement.

## Evidence Use Rules

Use Terraform-universal statements for behavior portable across execution
platforms. Attach execution-profile labels only to the selected apply model.
Attach platform labels only when the named platform owns the control.

Gold-standard examples describe observed internal implementation. They do not
approve a policy, prove configured repository settings, or establish a
universal requirement. A practice recommendation becomes mandatory only after
its governance owner approves it.

### Interpretation Rules

* Use a HashiCorp link for Terraform and HCP Terraform behavior, then state
	exactly which behavior the link supports.
* Use an internal source path for observed lessons, templates, workflows, or
	asset research. Do not describe an internal file as a vendor default.
* Use a gold-standard example to explain a pattern, but retain platform
	settings, access controls, and approval assumptions as unverified unless
	their configuration is independently available.
* Use `[Practice recommendation]` for a suggested control that needs an owner
	decision. Do not change the label to a mandate during a documentation edit.
* Use `[Assumption]` when an input is expected but not independently checked.
	Use `[Gap]` when available evidence cannot support a normative claim.

### Common Misreadings

| Misreading | Why it is incorrect | Correct reading |
| --- | --- | --- |
| A saved plan proves the artifact was approved | Terraform defines apply semantics, not reviewer behavior | Approval is a platform or organization control |
| A GitHub workflow source proves all repositories use it | The source is a current implementation example | Its scope is the observed repository pattern |
| HCP Terraform supports policies, so policy sets are mandatory | The service feature is documented | Assignment and enforcement remain organization decisions |
| A lesson uses AzureRM state, so AzureRM is required | The lesson is a teaching example | Backend choice remains DR-01 |
| A gap can be closed by a reasonable recommendation | Recommendation is not approval evidence | The assigned owner must publish a decision |

### Evidence Review Checklist

* The source can be opened by another reviewer
* The evidence label matches the source classification
* The claim does not exceed what the source proves
* A platform-specific detail does not appear in portable guidance
* An unresolved policy remains labeled as a gap or assumption
* The claim links to a guideline page where a reader can apply it

## Related Reference

Use the [glossary](glossary.md) for shared terminology and the
[known gaps and assumptions](known-gaps-and-assumptions.md) register before
stating an unresolved policy as a requirement.

Use [the library index](../index.md) to choose a route, and use
[the glossary](glossary.md) to resolve terms before comparing evidence records.