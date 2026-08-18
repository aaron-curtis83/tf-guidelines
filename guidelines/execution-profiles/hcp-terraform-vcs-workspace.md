---
title: HCP Terraform VCS Workspace Profile
description: Use HCP Terraform workspace runs when version control drives Terraform execution
---

## Profile Boundary

[Execution profile] Select this profile when an HCP Terraform workspace is
connected to version control and owns Terraform runs for its configuration and
state boundary. The workspace run model, policy checks, approvals, and state
handling are HCP Terraform responsibilities.

## VCS Run Lifecycle

[Execution profile] A pull request can produce a speculative plan for review.
After merge, the VCS workspace creates a fresh normal run from the merged
configuration. It does not apply the speculative pull-request plan.

Review the normal run according to the workspace's configured approval and
policy behavior before an apply proceeds. Retain the run and policy outcomes as
the delivery evidence for the merged configuration.

## State and Ownership

Assign each independently deployable root to the intended workspace and state
boundary. Keep workspace ownership, variable management, policy sets, and
approval responsibilities explicit before handing the configuration to an
operator.

The organization has not yet approved a universal HCP Terraform edition,
workspace naming convention, retention period, or approval policy. Record the
chosen settings as local delivery decisions until the relevant owner publishes
an organization baseline.

## Platform Controls

Use the [HCP Terraform overlay](../platform-overlays/hcp-terraform.md) for
workspace, run, policy, and approval behavior. Do not import saved-plan
artifact selection, GitHub environments, or GitHub reusable-workflow mechanics
into this profile.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable evidence expectations and
[handover evidence packs](../core/handover-evidence-pack.md) for operator
handover records.