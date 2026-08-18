---
title: Azure DevOps Delivery Overlay
description: Azure DevOps pipeline controls for Terraform delivery without GitHub workflow assumptions
---

## Scope

[Azure DevOps] Use this overlay when Azure DevOps Pipelines hosts a
runner-managed Terraform delivery process. Select the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md)
when the pipeline saves and later applies an exact binary plan.

## Pipeline Controls

[Azure DevOps] Define a pipeline stage or job boundary for Terraform
validation, planning, and applying. Scope pipeline identity, service
connections, variables, and secure files to the configuration and state
boundary they support. Serialize operations that can change the same state.

[Azure DevOps] Use Azure DevOps branch policies, pipeline checks, and
environment approvals when those controls are configured for the repository
and deployment target. Their required reviewers, checks, retention, and bypass
rules are organization-specific settings, not portable Terraform behavior.

## Plan and Apply Evidence

[Azure DevOps] When a pipeline applies a saved plan, retain the plan artifact,
configuration revision, validation outcome, and state coordinates. Verify the
selected artifact before apply and do not replace it with a new plan in a
later stage unless the delivery process deliberately returns to review.

The exact Azure DevOps artifact naming, retention, approval, identity, and
environment strategy is not established by the available evidence. Record
those settings as delivery gaps and obtain the owning organization's policy
before presenting them as a baseline.

## Destructive Operations

[Azure DevOps] Keep destruction separately initiated, explicitly authorized,
and serialized by the affected state boundary. Use the configured Azure DevOps
approval and environment controls for the destructive stage where available.

Do not describe GitHub reusable workflows, GitHub Environments, GitHub OIDC
permissions, or GitHub artifact-selection mechanics as Azure DevOps behavior.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable evidence expectations and
[state and boundaries](../core/state-and-boundaries.md) to identify the
affected state boundary.