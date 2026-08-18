---
title: Select an Execution Profile
description: Choose the Terraform execution authority before following platform-specific delivery guidance
---

## Choose One Apply Authority

Select one profile for each independently deployable configuration before
following a platform overlay. The profile determines how a reviewed change,
apply action, and delivery evidence relate to one another.

| When the configuration is delivered by | Select this profile |
| --- | --- |
| A CI runner that saves and later applies a reviewed binary plan | [Runner-managed reviewed plan](runner-managed-reviewed-plan.md) |
| An HCP Terraform workspace connected to version control | [HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md) |
| A CI runner that applies a saved binary plan | [Runner-managed reviewed plan](runner-managed-reviewed-plan.md) |
| An HCP Terraform workspace that creates a new run from the merged configuration | [HCP Terraform VCS workspace](hcp-terraform-vcs-workspace.md) |

## Decision Questions

Answer these questions before choosing an overlay:

1. Which service or runner owns the apply authority?
2. Does the selected model apply an exact saved binary plan, or does it create
   a new run from the merged configuration?
3. Which state boundary, identity, and approval controls govern that model?

Do not combine profiles for the same state boundary. A platform can host a
runner-managed workflow without changing it into an HCP Terraform workspace.

## Then Choose an Overlay

Use the applicable overlay after selecting the profile:

* [GitHub Actions](../platform-overlays/github-actions.md)
* [Azure DevOps](../platform-overlays/azure-devops.md)
* [HCP Terraform](../platform-overlays/hcp-terraform.md)

## Route Connections

Return to the [bespoke module authoring route](../routes/bespoke-module-authoring.md),
[published module upgrade route](../routes/published-module-upgrade.md), or
[client handover route](../routes/client-handover.md) when recording the
selected delivery context.