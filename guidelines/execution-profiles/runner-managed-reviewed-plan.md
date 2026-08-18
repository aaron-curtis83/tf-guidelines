---
title: Runner-Managed Reviewed Plan Profile
description: Use an exact saved Terraform plan when a CI runner owns planning and applying
---

## Profile Boundary

[Execution profile] Select this profile when a CI runner creates a binary
Terraform plan for a configuration and later applies that same plan. The
runner, not a Terraform workspace service, owns artifact retention, selection,
identity use, and apply orchestration.

## Preserve Plan Provenance

Associate the saved plan with the configuration revision, the state boundary,
and the successful validation run that created it. Before apply, retrieve the
intended artifact and verify that it contains the expected binary plan for the
same configuration and state coordinates.

[Execution profile] Applying a saved plan proves only that the runner selected
and applied that artifact. It does not prove that a reviewer approved it, that
branch protection passed, or that an independent deployment approval occurred.
Those are separate governance controls.

## Apply and Destructive Changes

Apply the exact saved binary plan rather than replacing it with a fresh plan
in the target branch. Keep lifecycle operations that affect the same state
boundary serialized. Treat destruction as a distinct, manually initiated
operation with its own reviewed plan and authorization controls.

## Platform Controls

Use the platform overlay for runner-specific mechanisms:

* [GitHub Actions](../platform-overlays/github-actions.md) for reusable
  workflows, artifacts, environments, and GitHub concurrency controls
* [Azure DevOps](../platform-overlays/azure-devops.md) for Azure DevOps
  pipelines, approvals, and deployment controls

Do not apply this profile to an
[HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md). That profile
creates a new run from the merged configuration instead of applying a
pull-request speculative plan.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
to define review evidence and [state and boundaries](../core/state-and-boundaries.md)
to identify the serialized state boundary.