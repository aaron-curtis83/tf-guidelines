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

## Service Connections and Resource-Owned Controls

[Azure DevOps] A service connection is standing access to an external system.
Use a named Azure Resource Manager service connection for each access boundary,
then authorize individual pipelines rather than granting all pipelines access.
Separate plan and apply connections when their Azure RBAC scope or risk differs.

| Operation | Connection scope | Pipeline authorization | Verification |
| --- | --- | --- | --- |
| PR plan | Provider reads and required backend access | Authorize PR pipeline explicitly | Can initialize and plan but cannot change managed resources |
| Merged apply | Deployment and state-write scope for one root | Authorize protected apply pipeline explicitly | Cannot deploy another root without a separate assignment |
| Manual destroy | Destructive scope for one state boundary | Authorize destroy pipeline explicitly | Plan pipeline cannot invoke destructive connection |

[Azure DevOps] A service connection cannot be supplied through a YAML variable.
Templates therefore need a visible static service connection reference in the
protected deployment stage, not a dynamically generated connection name.

```yaml
- task: AzureCLI@2
	inputs:
		azureSubscription: tf-payments-apply
		scriptType: bash
		scriptLocation: inlineScript
		inlineScript: terraform apply -input=false plan.tfplan
```

The connection name is original pseudocode, not an enterprise standard.
Verify the static connection matches the stage's root and state boundary.

[Azure DevOps] Environments and service connections own checks and approvals.
YAML can reference an Environment but cannot replace checks configured on the
Environment or service connection. Configure and record approvals, branch
control, required templates, business-hours checks, and exclusive locks on the
resource that owns the protected target or credential.

| Control need | Prefer ownership on | Evidence to record |
| --- | --- | --- |
| Deployment history and target approval | Environment | Environment name and check result |
| Credential-use approval | Service connection | Connection name and check result |
| Merge eligibility | Branch policy | Policy result before apply pipeline starts |
| Same-state exclusivity | Environment or service connection | Lock resource and deployment order |

[Gap] Approver groups, bypass rules, Environment names, connection naming, and
identity approach are organization decisions. Evidence must name the resource
and outcome, rather than claiming a YAML file approved a deployment.

## Stages and Pipeline Artifacts

[Azure DevOps] Separate validation, plan evidence, merged apply, and
post-apply validation so each control has a visible result.

| Stage | Trigger and purpose | Required output or gate |
| --- | --- | --- |
| PR validation | Pull request | Format, validation, tests, root selection, and source revision result |
| PR plan artifact | Pull request | Binary plan, rendered plan, manifest, checksum, and producing run ID |
| Merged apply | Protected branch after merge | Identified plan artifact passes verification before apply |
| Post-apply validation | Apply completion | Targeted validation, outputs or health checks, and delivery evidence |

[Azure DevOps] Publish binary plan, rendered plan, and manifest together as a
Pipeline Artifact. Artifact name and retention are local choices. The manifest
must identify root path, state identifier, source revision, producing run,
Terraform version, artifact name, and SHA-256 checksum.

```yaml
- publish: $(Build.SourcesDirectory)/evidence
	artifact: reviewed-tfplan

- download: current
	artifact: reviewed-tfplan
```

`download: current` retrieves an artifact from an earlier job or stage in the
same pipeline run. It is not enough when merged apply uses an artifact from an
identified producing PR run. Use a pipeline resource or
`DownloadPipelineArtifact@2` with producing pipeline and run identifiers,
then validate the downloaded manifest before apply.

```yaml
- task: DownloadPipelineArtifact@2
	inputs:
		buildType: specific
		project: $(System.TeamProject)
		pipeline: terraform-pr-plan
		runId: $(ProducingRunId)
		artifact: reviewed-tfplan
		path: $(Pipeline.Workspace)/reviewed-tfplan

- bash: |
		verify_manifest --revision "$MERGED_SHA" --root "$ROOT_PATH" \
			--state "$STATE_IDENTIFIER" --plan plan.tfplan
		terraform apply -input=false plan.tfplan
	displayName: Verify and apply selected plan
```

`ProducingRunId` must result from the repository's PR-to-merged-commit
selection process. Do not choose the latest successful artifact by pipeline
definition alone. The verification pseudocode must fail when revision, root,
state identifier, Terraform version policy, artifact name, or checksum differs.

## Lock Behavior and Saved-Plan Checks

[Azure DevOps] Choose `lockBehavior` deliberately for the state boundary.

| Choice | Use when | Effect to verify |
| --- | --- | --- |
| `sequential` | Every requested same-state apply must run in order | Later run waits and does not discard an earlier deployment |
| `runLatest` | Superseded deployments are explicitly safe to discard | Only newest queued run proceeds after lock release |

```yaml
lockBehavior: sequential
```

Neither is an enterprise default. `sequential` is normally easier to reason
about for mutable state. `runLatest` needs an explicit assessment that skipping
an older request does not omit required infrastructure or evidence. Pipeline
locking complements but does not replace Terraform backend locking.

[Execution profile] Before applying a saved plan, require all these checks:

* Producing run is the identified successful PR-plan run for merged PR and root
* Manifest source revision matches the merged commit under review
* Manifest root path and state identifier match protected stage inputs
* Artifact name, Terraform version policy, and SHA-256 checksum match
* Backend initialization targets the same state before `terraform apply plan.tfplan`

Stop and return to PR planning when a check fails. Do not create a fresh plan
inside merged apply to make an artifact mismatch pass. A Pipeline Artifact
associates bytes with a run; the manifest and validation establish provenance.
Neither is approval by itself.

## Destroy and Operational Controls

[Azure DevOps] Keep destroy manually initiated, root-scoped, and separately
authorized. Use a dedicated stage and connection when practical, a confirmed
state identifier input, a saved destroy plan, configured resource checks, and
a post-destroy evidence record.

For drift, run an identifiable non-mutating plan or refresh-oriented review
with a read-appropriate connection. Record root, state identifier, source
revision, connection, command mode, and result. Do not apply drift output
automatically.

[Gap] Enterprise service-connection names, artifact retention, approvals, and
recovery procedures are not established by available evidence. Keep them as
policy-owned records rather than inferring a GitHub Actions pattern.

## Related Guidance

Use [testing and change assurance](../core/testing-and-change-assurance.md)
for portable evidence expectations and
[state and boundaries](../core/state-and-boundaries.md) to identify the
affected state boundary.