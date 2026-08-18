---
title: GitHub Actions Delivery Overlay
description: GitHub Actions controls and internal examples for runner-managed Terraform delivery
---

## Scope

[GitHub Actions] Use this overlay only with the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md).
It records verified GitHub workflow-contract examples. It does not prescribe an
organization-wide GitHub baseline.

## Template Adoption

[GitHub Actions] [Gold-standard example] The module template is a reusable
module scaffold. Generated modules replace placeholder assertions and configure
validation, unit testing, and optional integration testing. The configuration
template is an independently deployable configuration scaffold. Its roots own
backend coordinates, non-sensitive values, and state; they do not use
cross-root references.

Template adoption remains descriptive. Current template README guidance refers
to reusable workflows at `@issue-20`, while current callers use `@v1`. The
configuration README also names `TERRAFORM_DEPLOY_AZURE_CLIENT_ID`, while the
apply caller uses `TERRAFORM_APPLY_CLIENT_ID`. Do not treat either documented
name as an approved adoption contract until the discrepancy is reconciled.

## Reusable Workflow Contract

[GitHub Actions] [Gold-standard example] Callers invoke versioned
`workflow_call` workflows, pass named `with` inputs, and map named `secrets`.
Configuration lifecycle callers provide the working directory, AzureRM backend
resource group, storage account, container, state key, and optional variable
file or artifact name. Plan exposes `has-changes`; release receives a
title-derived semantic version and target merge commit.

Record required and optional inputs with the selected reusable-workflow version
rather than duplicating workflow YAML. The current internal callers reference
the workflow library at `@v1`; that is an example, not an organization-wide
versioning policy.

## Identity and State Controls

[GitHub Actions] [Gold-standard example] Plan and drift use a read-oriented
OIDC client identity, apply uses a deployment identity, and destroy uses a
destructive identity. The workflows request `id-token: write`, use
`Azure/login@v3`, and set `ARM_USE_OIDC`.

[GitHub Actions] [Gold-standard example] A lowercase-kebab workload slug and
environment slug derive a state key of
`<workload-slug>-<environment-slug>.tfstate`. Lifecycle operations serialize
by state key. Keep one state per independently deployable root.

[Practice recommendation] Restrict federated identity claims and RBAC to the
configuration and state scopes. A shared configuration identity is an observed
organization baseline; separate identities by operation are a higher-assurance
option, not a universal requirement.

## Saved Plan Provenance

[GitHub Actions] [Gold-standard example] Pull-request workflows plan changed
roots, save `terraform.tfplan` and rendered evidence under
`.terraform-workflow`, and upload a named plan artifact with 14-day retention.
After merge, the apply workflow selects a successful pull-request workflow run
whose `head_sha` equals the merged pull request head SHA. It downloads the
named artifact from that exact run, verifies `terraform.tfplan`, initializes
the matching backend, and applies the saved binary plan without creating a
replacement plan on `main`.

This provenance chain does not establish reviewer approval, an immutable or
attested artifact, or an independent apply approval. Branch protection,
pull-request review, and environment protection are separately configured
GitHub controls.

## Destroy, Quality, and Release

[GitHub Actions] [Gold-standard example] Destroy is manually initiated,
root-scoped, state-key serialized, and based on a saved destroy plan. The
workflow targets the `terraform-destroy` Environment. Its reviewer protection
is not verified from source.

[Practice recommendation] Repositories that manage deployed Terraform state
should protect `terraform-destroy` with an approver other than the initiator
and no administrator bypass. Apply environment protection remains a
risk-based local choice rather than a default requirement.

[GitHub Actions] [Gold-standard example] Module CI classifies pull-request
titles and can run validation, unit tests, documentation checks, and optional
integration tests. Releases run for merged pull requests to `main`, validate
the semantic version and merge commit, publish a GitHub Release, and update a
major compatibility tag for non-release-candidate releases.

Release-tag protection, signing, and supply-chain review settings are not
verified. Treat them as configuration gaps until their owning policy is
published.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable assurance requirements and the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md)
for the cross-platform plan model.