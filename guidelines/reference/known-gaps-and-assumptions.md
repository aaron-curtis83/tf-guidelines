---
title: Known Gaps and Assumptions
description: Unresolved Terraform practice policy and validation decisions
---

## Decision Register

The topics in this register have insufficient local evidence for a normative
organization rule. Keep related guidance descriptive until the named owner
records an approved disposition.

| ID    | Label                    | Topic                                      | Owner needed                    | Impact | Disposition                                                                                     |
|-------|--------------------------|--------------------------------------------|---------------------------------|--------|-------------------------------------------------------------------------------------------------|
| DR-01 | `[Gap]`                  | Approved state backend and access standard  | Terraform platform governance   | High   | Do not prescribe a backend, encryption, access, or locking baseline                            |
| DR-01 | `[Gap]`                  | Approved module registry and publication model | Terraform platform governance | High   | Do not require a registry, Git source, release gate, or publication approval model              |
| DR-01 | `[Gap]`                  | Supported HCP Terraform edition             | HCP Terraform service owner     | High   | Do not prescribe edition features, workspace naming, or policy-set availability                 |
| DR-01 | `[Gap]`                  | Plan and delivery-evidence retention        | Records and platform governance | High   | Do not declare the observed 14-day GitHub artifact retention an organization standard           |
| DR-01 | `[Gap]`                  | State and artifact recovery                 | Operations and platform owner   | High   | Define backup, break-glass, stale-plan, expired-artifact, and reconciliation procedures         |
| DR-01 | `[Gap]`                  | Handover accountability                     | Delivery and operations owner   | High   | Assign backend, workspace, identity, approval, policy, and recovery contacts                    |
| DR-02 | `[Assumption]`           | Independent child-module consumption and release | Module governance owner      | Medium | Keep nested children bundled with their parent; do not promise independent release streams      |
| DR-03 | `[Gap]`                  | Markdown, frontmatter, and link validation tooling | Documentation platform owner | Medium | Use manual checks for this release; select and configure automated validation before publication |
| DR-04 | `[Gap]`                  | Route usability validation                  | Guideline product owner         | Medium | Pending human validation: delivery leader authors a bespoke module, contributor upgrades a published module, and client operator locates handover evidence |
| DD-02 | `[Gap]`                  | GitHub template contract drift               | Template and workflow owners    | High   | Documentation and callers were not verified in the restricted executable-source review. Inspect both before publishing a workflow reference or secret-name contract |
| DD-05 | `[Gap]`                  | GitHub executable-source discovery           | Template and workflow owners    | High   | Only a reusable validation workflow and disabled PR-plan and merged-apply references were reviewed. Discover enabled callers and their platform configuration before declaring a broader GitHub baseline |

### Register Interpretation

A register entry prevents an unsupported policy claim. It is not a reason to
stop independent work that can proceed within the available evidence. For
example, an upgrade can compare module inputs and plan impact while the module
registry model remains unresolved.

| Label | Use when | Required response |
| --- | --- | --- |
| `[Gap]` | Evidence cannot support a normative rule | Name the owner, affected guidance, and interim boundary |
| `[Assumption]` | A needed input is expected but not independently validated | State the validation needed and prevent it becoming a mandate |

### Reader Actions by Register Area

| Register area | Route impact | Interim action |
| --- | --- | --- |
| DR-01 backend and recovery | All routes; especially handover and state migration | Record current state owner and do not prescribe a backend baseline |
| DR-01 registry and support | Bespoke authoring and published upgrades | Record observed source and version; defer publication policy |
| DR-01 HCP Terraform policy | HCP workspace delivery and handover | Record workspace facts; do not prescribe edition or policy-set assignment |
| DR-01 retention | Runner-managed evidence and handover | Link current evidence; do not declare retention duration |
| DR-02 nested-module consumption | Bespoke authoring | Keep child modules bundled with parent unless governance decides otherwise |
| DR-03 documentation checks | Every changed guideline page | Use manual frontmatter, link, label, and diagnostic checks |
| DR-04 route usability | All routes | Schedule human task walkthroughs before declaring usability validated |
| DD-02 template drift | GitHub Actions overlay | Keep adoption instructions descriptive until documentation and executable callers are reviewed together |
| DD-05 executable-source discovery | GitHub Actions overlay | Treat local examples as bounded references until enabled callers and platform configuration are independently reviewed |

### Original Escalation Record

Use this original record when a route finds an unapproved operational decision:

```text
register_item: DR-01
label: [Gap]
topic: State recovery for application-connectivity-production
affected_guidance: client-handover and state-and-boundaries
known_fact: Current root path and state owner are recorded
decision_needed: Recovery owner, procedure, and evidence location
interim_boundary: Do not claim a recovery standard
owner: Operations and platform owner
```

The record makes the limitation measurable. It does not fabricate a recovery
procedure or require a particular backend.

## GitHub Template Drift

[Gap] The restricted executable-source review did not verify GitHub template
READMEs, enabled callers, a workflow-version reference, or a template-specific
apply-client secret name. Do not publish an adoption directive from README text
or infer that a caller is current without inspecting it.

The available executable sources demonstrate only a reusable validation
workflow plus disabled PR-plan and merged-apply references. They use
`AZURE_PLAN_CLIENT_ID` and `AZURE_DEPLOY_CLIENT_ID` in those references, but
that observation does not establish a published template contract.

### Template Drift Verification

Before changing this gap, inspect both the documentation and the executable
caller or workflow source. Record the exact reference, secret name, repository
path, revision reviewed, and whether the workflow is enabled. Do not close the
gap because one source changes in isolation.

| Check | Evidence needed | Do not infer |
| --- | --- | --- |
| Workflow reference | README and current caller agree on a supported reference | That tags are protected or signed |
| Apply-client secret | README and caller agree on the supported name | That the identity claim and RBAC are approved |
| Reusable workflow inputs | Contract documentation matches executable interface | That all callers have adopted the new interface |
| Environment controls | Platform configuration is independently verified | That workflow YAML proves reviewer rules |

## Pending Human Route Validation

The following route checks remain pending and were not simulated as user tests
for this release:

1. A delivery leader follows the bespoke-module authoring route to define and
	deliver a bespoke Terraform module.
2. A contributor follows the published-module upgrade route to complete a
	compatibility, state-impact, and delivery-evidence decision.
3. A client operator follows the client handover route to locate the delivery
	evidence, execution context, and ownership decisions needed to assume
	responsibility.

Record missed steps, ambiguous language, and unresolvable links with the
Guideline product owner before closing DR-04.

### Route-Test Evidence Record

Use one record for each participant and route. The test is complete only when
the participant can independently find the needed pages and completion record.

```text
participant_role: client operator
route: client-handover
task: Locate the apply authority and evidence for payments-production
outcome: pending
missed_steps: none recorded
ambiguous_terms: pending participant feedback
unresolvable_links: pending participant feedback
owner: Guideline product owner
```

Do not replace user-route testing with an author review. Manual link checking
can confirm navigation exists, but only route participants can validate whether
the task sequence is understandable.

### Gap Review Sequence

1. Identify the specific route, core page, profile, or overlay affected.
2. Separate facts supported by sources from the decision that remains open.
3. Name the accountable owner and the evidence needed for a decision.
4. State the interim boundary in neutral language.
5. Link the register item from the affected guidance.
6. Review the item when the owner publishes evidence or a corrected contract.

### Failure Modes

| Failure mode | Consequence | Correction |
| --- | --- | --- |
| A gap is hidden in generic wording | Readers assume a baseline exists | Add the label, owner, and interim boundary |
| A recommendation is written as a rule | Policy is fabricated | Restore `[Practice recommendation]` or `[Gap]` |
| A local template is treated as a platform default | Platform behavior is overstated | Cite the internal source and its observed scope |
| An owner is missing | The decision cannot progress | Assign the responsible policy or service owner |
| A closure has no evidence | Future readers cannot audit the decision | Link the approved policy or corrected repository source |

## Resolution Evidence

Close a register entry only when its owner publishes an approved policy or a
verified repository correction. Record the decision, effective date, scope,
and replacement guidance before converting an assumption or gap into a
requirement.

### Closure Record

Close an item with a dated record that another reader can verify:

```text
register_item: <ID>
decision_owner: <role or team>
evidence: <approved policy or corrected repository location>
effective_date: <YYYY-MM-DD>
scope: <affected roots, modules, or platforms>
guidance_updated: <pages updated after approval>
```

Closing an item changes the evidence boundary only for the recorded scope. It
does not automatically resolve related gaps, such as state recovery and
artifact retention, unless the approved decision explicitly covers both.

### Register Verification Checklist

* Every open item has a label, topic, owner, impact, and interim disposition
* Affected route, core, profile, or overlay pages link to the relevant gap
* No gap is described as an approved organization requirement
* GitHub template drift remains descriptive until sources are reconciled
* Human route validation remains open until participant records are complete
* A closed item links to independently reviewable approval evidence

## Related Reference

Use the [evidence catalog](evidence-catalog.md) to select a claim label and
the [GitHub Actions overlay](../platform-overlays/github-actions.md) for the
bounded GitHub delivery description.

Return to [the library index](../index.md) to select an independent task that
can proceed while an owner resolves a policy or template gap.