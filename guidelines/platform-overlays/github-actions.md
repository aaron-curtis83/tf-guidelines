---
title: GitHub Actions Delivery Overlay
description: Executable-source-bounded GitHub Actions guidance for Terraform validation and reviewed plans
---

## Scope

[GitHub Actions] Use this overlay with the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md)
only when a repository implements that model. The local sources demonstrate a
reusable validation workflow and two disabled template-reference workflows.
They are examples of workflow mechanics, not an enabled deployment baseline or
an organization-wide GitHub Actions contract.

> [!IMPORTANT]
> Executable workflow YAML is the local behavioral evidence for this overlay.
> README text can identify a documentation discrepancy, but cannot establish
> delivery behavior.

## Reusable Validation

[GitHub Actions] [Gold-standard example] The reusable validation workflow
accepts one required string input named `working-directory`. It grants
`contents: read`, sets that input as the default working directory, then runs
`terraform fmt -check -recursive`, `terraform init -backend=false -input=false`,
and `terraform validate -no-color`.

The workflow skips template repositories with
`github.event.repository.is_template == false`. That condition and the
backend-free initialization keep validation focused on configuration syntax and
formatting. They do not validate remote backend access, provider credentials,
or a deployable plan.

```yaml
on:
  workflow_call:
    inputs:
      working-directory:
        required: true
        type: string

permissions:
  contents: read

jobs:
  validate:
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
```

[Practice recommendation] A caller should pass a reviewed Terraform
configuration directory, not a user-controlled arbitrary path. Add any
repository-specific root-selection rules to the caller or workflow contract
after their executable implementation is available.

## Disabled PR Plan Reference

[GitHub Actions] [Gold-standard example] The local PR-plan reference is
guarded by a literal `TEMPLATE_REFERENCE_ENABLED: false`. If a generated
configuration repository deliberately adapts it, the shown job runs only after
that repository changes the guard and establishes its own controls.

When enabled by such a repository, the reference workflow:

1. Runs for pull requests targeting `main`.
2. Checks out `github.event.pull_request.head.sha`, rather than an implicit
   branch tip.
3. Requests `contents: read` and `id-token: write`.
4. Uses `azure/login@v2` with plan client, tenant, and subscription values.
5. Runs `terraform plan -input=false -out=plan.tfplan`.
6. Renders that binary plan to `plan.txt`.
7. Calculates the SHA-256 checksum of `plan.tfplan`.
8. Uploads `plan.tfplan`, `plan.txt`, and `plan-manifest.json` in the named
   `reviewed-tfplan` artifact for 14 days.

`id-token: write` allows the job to request an OpenID Connect token. The source
does not provide the corresponding federated-credential claims or Azure RBAC,
so it does not prove the granted Azure access scope.

### Evidence Manifest

[Gold-standard example] The reference creates the following JSON document with
`jq` in the plan-producing job. These are the complete evidenced manifest
fields. Root selection, state identifiers, artifact names, and approval facts
are not manifest fields in the local reference.

```json
{
  "schemaVersion": 1,
  "prNumber": "<pull-request-number>",
  "headSha": "<pull-request-head-sha>",
  "baseSha": "<pull-request-base-sha>",
  "planRunId": "<plan-workflow-run-id>",
  "terraformVersion": "<terraform-version>",
  "planSha256": "<sha256-of-plan.tfplan>"
}
```

[GitHub Actions] [Gold-standard example] The artifact retention is 14 days in
this disabled reference. It is not an organization retention requirement.
The rendered text supports review, while the binary-plan checksum is the
mechanism used by the reference to detect a changed plan file.

## Disabled Merged Apply Reference

[GitHub Actions] [Gold-standard example] The merged-apply reference responds
to closed pull requests targeting `main`, then proceeds only for merged pull
requests and only while its template guard remains deliberately changed from
the supplied `false` value.

Its selection and verification sequence is specific and fail-closed:

1. Locate the workflow whose name equals `Template Reference PR Terraform Plan`.
2. Require exactly one workflow with that name.
3. Locate successful pull-request runs associated with the merged pull request.
4. Require exactly one candidate run.
5. Download the `reviewed-tfplan` artifact from that run into `reviewed-plan`.
6. Require `reviewed-plan/plan-manifest.json` and
   `reviewed-plan/plan.tfplan` to exist.
7. Verify `schemaVersion`, PR number, head SHA, base SHA, producing plan run
   ID, and the binary plan's SHA-256 against the manifest.
8. Authenticate with `azure/login@v2` using the deploy client-ID secret.
9. Initialize Terraform and run
   `terraform apply -input=false reviewed-plan/plan.tfplan`.

The source checks the manifest head SHA against the merged pull request head
SHA. It does not use a fresh post-merge plan. It also does not verify that
`plan.txt` exists during apply, even though the PR-plan reference uploads it.

```yaml
- name: Verify saved-plan provenance
  run: |
    test "$(jq -r .prNumber "$manifest")" = "$PR_NUMBER"
    test "$(jq -r .headSha "$manifest")" = "$HEAD_SHA"
    test "$(jq -r .baseSha "$manifest")" = "$BASE_SHA"
    test "$(jq -r .planRunId "$manifest")" = "$RUN_ID"
    test "$(sha256sum "$plan" | cut -d ' ' -f 1)" = "$(jq -r .planSha256 "$manifest")"

- name: Apply saved plan
  run: terraform apply -input=false reviewed-plan/plan.tfplan
```

[Execution profile] Applying the selected binary plan preserves the reviewed
plan input. It does not establish reviewer approval, artifact attestation,
or an independently configured deployment approval.

## Identity Boundary

[GitHub Actions] [Gold-standard example] The disabled references use two
different secret names for Azure login client IDs: `AZURE_PLAN_CLIENT_ID` for
the plan job and `AZURE_DEPLOY_CLIENT_ID` for the apply job. Both also use
tenant and subscription secrets. The source demonstrates distinct input names;
it does not establish a separation of Azure permissions, identity ownership,
or operation-specific RBAC.

[Gap] The local sources do not establish the federated identity's subject or
audience claims, Azure role assignments, `ARM_USE_OIDC`, an `@v3` Azure login
action, or a reusable workflow identity contract. Do not claim those controls
as locally implemented behavior.

[Practice recommendation] Before enabling an Azure-authenticated workflow,
document the job-to-identity mapping and verify its federated claims and Azure
permissions independently. The [GitHub OpenID Connect documentation](https://docs.github.com/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-azure)
is authoritative for configuring GitHub's Azure OIDC integration, not these
local template references.

## Controls Not Established Locally

[Gap] The following controls are not evidenced by the available executable
sources. A repository may implement them, but it must supply its own
executable evidence or approved policy before this guidance describes them as
local behavior.

| Control area | Not established by the sources |
| --- | --- |
| Reusable lifecycle contract | `workflow_call` plan or apply contracts, named lifecycle inputs or secrets, `@v1`, and `has-changes` outputs |
| State orchestration | Derived state keys, working-root selection beyond validation, changed-root detection, concurrency, and cancellation behavior |
| Lifecycle operations | Destroy, drift detection, module CI, release or tag automation, and post-apply checks |
| GitHub protection | GitHub Environments, reviewer protection, branch protection, artifact attestation, signing, and retention policy |
| Identity model | Operation-specific identity permissions beyond the distinct plan and deploy client-secret names |

## Adoption Checklist

When adapting the disabled references in a generated repository, record the
repository's choices rather than inheriting unstated assumptions:

* Remove or retain the template guard only through that repository's reviewed change process
* Identify the Terraform configuration directory and backend behavior
* Define the federated-credential and Azure RBAC evidence outside workflow YAML
* Preserve the manifest fields and saved-plan checksum checks when using the reviewed-plan model
* Decide artifact retention, reviewer controls, and deployment authorization with the owning policy or platform team
* Add executable tests for any root selection, concurrency, destroy, drift, release, or reusable-workflow behavior introduced by the repository

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable assurance requirements, the
[runner-managed reviewed plan profile](../execution-profiles/runner-managed-reviewed-plan.md)
for saved-plan semantics, and
[known gaps and assumptions](../reference/known-gaps-and-assumptions.md)
for unresolved GitHub template and platform decisions.