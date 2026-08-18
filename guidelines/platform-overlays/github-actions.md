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

## Workflow Contract

[GitHub Actions] Define the reusable workflow contract before a repository
adopts it. The caller supplies root and backend coordinates; the reusable
workflow owns execution and returns named outputs. Keep the contract visible in
the calling workflow so reviewers can inspect the requested state boundary and
identity path.

| Contract item | Caller supplies | Workflow verifies or produces | Evidence boundary |
| --- | --- | --- | --- |
| Root selection | `root_path` | Checked-out directory is a Terraform root | [Gold-standard example] |
| State selection | `state_identifier` and backend coordinates | Concurrency group and backend match selected state | [GitHub Actions] |
| PR revision | PR head SHA | Checkout uses that SHA, not a branch tip | [Gold-standard example] |
| Plan evidence | Artifact name | Binary plan, rendered plan, and manifest are uploaded together | [Gold-standard example] |
| Apply request | Merged PR and selected root | One verified candidate artifact is applied | [Execution profile] |
| Release request | Version and merge commit | Version syntax and commit relationship are checked | [Gold-standard example] |

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

### OIDC Verification

[GitHub Actions] Request only permissions required for the job that obtains an
Azure workload token. A job that checks out source and signs in with OIDC has
this minimum GitHub permission shape:

```yaml
permissions:
	contents: read
	id-token: write
```

`id-token: write` permits token retrieval. It does not grant Azure access. The
Azure federated credential needs at least one claim condition, and Azure RBAC
must restrict the exchanged identity to the configuration and state resources
it needs. For newly created, renamed, or transferred repositories, prefer a
stable repository-ID subject claim when the federated credential design permits
it. Review both token subject and Azure RBAC; a narrow subject with broad
subscription RBAC remains broad deployment access.

| Operation | Identity capability | Trust-condition considerations | Verification |
| --- | --- | --- | --- |
| PR plan | Provider reads and required state access | Restrict repository and plan-workflow subject | Can initialize and plan but cannot change managed resources |
| Merged apply | Deployment and state-write access | Restrict repository and protected branch or Environment subject | Can modify only assigned root scope |
| Manual destroy | Destructive access for selected root | Require separate protected workflow or Environment subject | Cannot be used by plan workflow |
| Drift detection | Read access for refresh and plan | Limit to drift workflow and target scope | Does not change resources |

### State-Key Concurrency

[GitHub Actions] Serialize operations by state boundary, not repository name
or workflow file. A state-key group prevents two runs for the same state while
allowing independent roots to proceed.

```yaml
concurrency:
	group: terraform-${{ inputs.state_identifier }}
	cancel-in-progress: false
```

Superseded PR validation or plan runs may be cancelled because a newer,
unreviewed commit supersedes them. Same-state apply and destroy must serialize
without cancelling an active run, because it may hold a backend lock or be
changing resources. GitHub concurrency limits scheduling; it does not replace
Terraform backend locking.

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

[Gold-standard example] The named artifact contains a binary plan, rendered
plan, and manifest. Fourteen-day retention is observed local behavior, not an
organization requirement. Produce the manifest in the same job as the plan and
compute the checksum after the binary plan is final.

```json
{
	"root_path": "roots/payments",
	"state_identifier": "payments-prod",
	"pull_request": 184,
	"head_sha": "<pr-head-sha>",
	"base_sha": "<pr-base-sha>",
	"producing_run_id": "<github-run-id>",
	"terraform_version": "1.9.8",
	"artifact_name": "reviewed-tfplan",
	"plan_sha256": "<sha256-of-plan.tfplan>"
}
```

The rendered plan supports review but does not replace checksum verification.
The manifest supplies provenance data; it does not prove approval.

### Plan Procedure

1. Check out the pull request's immutable head SHA.
2. Initialize the intended backend and record the Terraform version.
3. Validate and create the binary plan for the selected root and state.
4. Render the binary plan, calculate its SHA-256, and write the manifest.
5. Upload binary plan, rendered plan, and manifest as one named artifact.
6. Record workflow run URL, PR number, head SHA, state identifier, and checksum.

Stop when the job cannot identify one root, initialize the expected backend,
produce a binary plan, or upload the matching manifest.

### Merged Apply Selection and Verification

[Execution profile] An apply workflow must select one successful PR-plan
candidate, not "the latest successful run." Follow this exact algorithm:

1. Identify the merged pull request and its immutable head SHA.
2. Query successful PR-plan runs for that pull request and root.
3. Keep only candidates whose manifest head SHA matches the merged PR head SHA.
4. Require exactly one candidate. Stop on zero or more than one candidates.
5. Download the named artifact from that candidate's producing run ID.
6. Verify binary plan, rendered plan, and manifest are all present.
7. Verify PR number, head SHA, root path, state identifier, artifact name,
	 Terraform version policy, and binary SHA-256 against the manifest.
8. Initialize the backend for the same root and state identifier.
9. Run `terraform apply plan.tfplan` without creating a replacement plan.
10. Record candidate run ID, manifest checksum, apply run ID, and outcome.

Return to pull-request planning and review if any check fails. A stale or
ambiguous candidate cannot be repaired by generating a fresh plan during apply.

```yaml
- name: Verify selected plan
	run: |
		verify_manifest --pr "$PR_NUMBER" --head "$MERGED_HEAD_SHA" \
			--root "$ROOT_PATH" --state "$STATE_IDENTIFIER" \
			--artifact "reviewed-tfplan" --plan plan.tfplan

- name: Apply selected plan
	run: terraform apply -input=false plan.tfplan
```

`verify_manifest` is repository-owned pseudocode. It must fail closed on every
mismatch in the selection algorithm.

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

### Manual Destroy and Drift

[GitHub Actions] Run drift detection as a separately identifiable
read-oriented operation. Record root, state identifier, configuration commit,
identity, command mode, and result. Do not apply drift output automatically.
Use the delivery process to decide whether configuration or remote
infrastructure must change.

[Gold-standard example] Destroy is manually initiated, root-scoped,
state-key serialized, and based on a saved destroy plan. Use a confirmed state
identifier input, a separate destructive identity, a protected destroy
workflow or Environment where configured, and an evidence record. The current
`terraform-destroy` Environment pattern does not prove any reviewer setting.

* Confirm root and state identifier in both request and workflow input
* Produce, render, checksum, and retain a destroy plan before execution
* Serialize by state identifier without cancelling an active state run
* Record configured Environment or branch-control outcome
* Apply only the reviewed destroy plan and record remaining resources and state result

### CI and Release Evidence

| Change | Minimum CI evidence | Release evidence when applicable |
| --- | --- | --- |
| Module contract change | Formatter, validate, tests, and consumer plan | Version decision and compatibility note |
| Provider or Terraform constraint change | Constraint and lock-file review with representative tests | Minimum supported version and release notes |
| Root infrastructure change | Rendered plan, manifest, and selected apply evidence | Not applicable unless root publishes an interface |
| Release candidate | CI result and target commit | Candidate version, commit, release URL, and tag decision |

[Gap] Release-tag protection, signing, artifact attestation, Environment
reviewers, and supply-chain review settings are not verified. Do not infer them
from a successful workflow run.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable assurance requirements and the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md)
for the cross-platform plan model.