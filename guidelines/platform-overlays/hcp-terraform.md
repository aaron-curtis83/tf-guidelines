---
title: HCP Terraform Delivery Overlay
description: HCP Terraform workspace, run, policy, and approval behavior for VCS-driven configurations
---

## Scope

[HCP Terraform] Use this overlay with the
[HCP Terraform VCS workspace profile](../execution-profiles/hcp-terraform-vcs-workspace.md).
HCP Terraform owns the workspace run lifecycle; GitHub Actions and Azure
DevOps runner mechanics do not apply to that lifecycle.

## Workspace and Run Behavior

[HCP Terraform] Associate the VCS-connected configuration with its intended
workspace and state boundary. A pull request can create a speculative plan for
review. After merge, HCP Terraform creates a fresh normal run from the merged
configuration.

[HCP Terraform] Do not apply a pull-request speculative plan after merge and
do not treat it as a saved runner-managed plan artifact. Review the normal run
that represents the merged configuration before the configured apply action.

## Policy and Approval Behavior

[HCP Terraform] Workspace policy checks and configured run approvals can
govern whether a run may apply. Record the workspace, run, policy, and approval
outcomes as delivery evidence for the merged configuration.

The organization has not established a universal HCP Terraform edition,
policy-set assignment, approval requirement, workspace naming convention, or
retention setting. Treat these as configuration gaps pending an approved
organization policy.

## Boundary Preservation

Do not import GitHub reusable-workflow inputs, GitHub plan artifacts, GitHub
Environments, Azure DevOps pipeline stages, or runner-managed saved-plan
provenance into an HCP Terraform VCS workspace.

## Related Guidance

Use the [HCP Terraform VCS workspace profile](../execution-profiles/hcp-terraform-vcs-workspace.md)
for execution semantics and
[testing and change assurance](../core/testing-and-change-assurance.md) for
portable review evidence.