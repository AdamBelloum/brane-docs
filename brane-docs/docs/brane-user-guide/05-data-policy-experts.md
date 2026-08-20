# Brane Policy Manager Guide

**Audience:** Policy Managers responsible for maintaining eFLINT policy versions for a Brane worker domain.

This guide describes the policy-management workflow supported by `brane-deployment` baseline `369392b991e0c3290739077d0ad071b5ce3f76bb`.

A Policy Manager prepares, uploads, inspects, and explicitly activates domain policies. Administrators generate policy-manager tokens and maintain infrastructure. Users submit workflows and do not administer policy versions.

## Policy Manager responsibilities

A Policy Manager is responsible for:

1. preparing a domain-appropriate `.eflint` policy file;
2. obtaining a valid policy-manager token from an Administrator;
3. selecting the intended worker domain;
4. uploading a policy as a new version;
5. inspecting available versions and active state;
6. explicitly activating the intended version;
7. diagnosing policy upload, activation, and denial outcomes.

A Policy Manager is **not** responsible for deploying infrastructure, generating policy-manager tokens, distributing certificates, building datasets, or submitting user workflows.

## 1. Understand the policy lifecycle

Use this lifecycle for every policy change:

```text
prepare
  → inspect current state
  → upload a new version
  → inspect available versions
  → confirm and activate one version
  → inspect active state
  → verify behaviour with a controlled workflow
```

Uploading a policy creates a version on the selected worker domain. It **does not activate** that version.

Activation changes the policy used for subsequent policy decisions. Treat activation as a deliberate production change: inspect the version identifier, confirm the target domain, and record the reason for the change.

## 2. Required inputs

Before starting, obtain:

| Input | Source | Purpose |
|---|---|---|
| A `.eflint` policy file | Policy Manager | The policy source to upload. |
| Policy-manager token file | Administrator | Authorises policy operations for one domain and a limited period. |
| Target worker domain | Inventory or Administrator | Identifies where the policy checker runs. |
| SSH access to the worker | Administrator-managed access | Used by the shell helper for policy operations. |
| Expected workflow/data identities | User and domain stakeholders | Ensures policy rules refer to the workflow, package, data, and domain identifiers actually in use. |

Store policy source files under the deployment repository's `policies/` directory where practical. The current repository includes `policies/my-policy.eflint` as a local policy source file.

A policy-manager token is secret material. Store the supplied token file in:

```text
policy_tokens/
```

Never commit token files, paste token contents into documentation, or share a token through an unapproved channel.

## 3. Prepare a policy change

Before uploading a policy:

1. Identify the worker domain where the policy will apply.
2. Confirm the workflow, package, dataset, user, and domain identifiers that the policy must govern.
3. Review the intended allow and deny cases with the responsible domain stakeholders.
4. Keep the policy file as `.eflint`.
5. Decide how the new version should differ from the currently active version.
6. Define a controlled verification case for after activation.

The deployment begins with **deny-all** behaviour. Therefore, a remote workflow may be submitted successfully and reach the worker infrastructure, yet still be denied during policy evaluation. This is expected until an applicable policy version has been uploaded and activated.

Do not treat a policy denial as proof of an infrastructure failure. First inspect the active policy state and whether the policy permits the requested operation.

## 4. Preferred workflow: Policy Management workspace

The Streamlit **Policy Management** workspace is the preferred interface because it keeps token values out of task metadata and ordinary task logs.

### 4.1 Select context

1. Open **Policy Management**.
2. Select a valid policy-manager token file.
3. Select the target worker domain.
4. Run **Check SSH** when connectivity has not recently been confirmed.
5. Confirm that the displayed token status is valid and has sufficient remaining lifetime.

The workspace displays token filenames and expiry status, not token values.

### 4.2 Inspect policy state

Before changing policy:

1. Open the **Status** area.
2. Select **Inspect policy state**.
3. Wait for the policy-list task to complete.
4. Record the current active version and relevant available versions.

Use **Task History** if a task output is no longer visible in the current browser session.

### 4.3 Upload a new policy version

1. Open **Upload policy**.
2. Select the local `.eflint` file.
3. Confirm the selected worker domain and token context.
4. Select **Add policy version**.
5. Wait for the upload task to complete.
6. Record the version identifier returned by the task.

Uploading adds a version but leaves the previous active policy unchanged.

### 4.4 Inspect versions after upload

1. Open **Activate policy**.
2. Select **List available versions**.
3. Compare the newly returned version identifier with the upload result.
4. Confirm that the intended version is available for the intended worker domain.

Do not activate a version merely because it is the newest version. Confirm its purpose and expected effect.

### 4.5 Activate deliberately

1. Enter the exact policy version identifier.
2. Confirm the intended worker domain.
3. Select the activation-confirmation checkbox.
4. Select **Activate policy version**.
5. Wait for the activation task to finish.
6. Inspect policy state again and confirm that the expected version is active.

The dashboard performs activation as a separate task and verifies active state afterward.

## 5. Shell-helper workflow

Use the interactive helper only from a trusted administrator-managed workstation where SSH access to worker nodes is configured:

```sh
cd /path/to/brane-deployment

bash scripts/brane_helper_policy.sh
```

The helper supports:

- environment and token-expiry checks;
- selecting a token file from `policy_tokens/`;
- selecting a worker from the Ansible inventory;
- uploading and adding a local `.eflint` file;
- listing available policy versions;
- activating a selected version.

The helper invokes policy commands on the worker's local checker endpoint. Do not copy its generated commands into documentation or shell-history examples because policy credentials must not be exposed.

The underlying command families are:

```text
branectl policies add <INPUT> [--language eflint]
branectl policies list
branectl policies activate [VERSION]
```

These operations require valid policy-manager credentials and connectivity to the target worker checker.

## 6. Token handling and expiry

Administrators issue domain-specific, time-limited policy-manager token files. If no usable token is available:

1. do not attempt to create one from a policy-manager workstation;
2. request a replacement from the Administrator;
3. specify the policy-manager identity, target domain, and required validity period;
4. store the received file in `policy_tokens/` with owner-only permissions where supported;
5. remove expired or superseded token files according to local credential-retention practice.

A token may be structurally readable but lack the required authority or be expired. In both cases, request a new token from the Administrator rather than weakening policy controls.

## 7. Policy troubleshooting reference

| Symptom | Likely cause | First action |
|---|---|---|
| No token files appear | Token was not received or was stored outside `policy_tokens/`. | Obtain the correct token file from the Administrator and store it securely. |
| Token is expired or invalid | Token lifetime elapsed, file is corrupted, or it is for another domain. | Request a replacement token; do not edit token contents. |
| SSH check fails | Worker host, SSH user, network path, or access configuration is incorrect. | Confirm the selected inventory worker and ask the Administrator to verify SSH access. |
| Upload fails before a version is returned | Invalid policy path, remote connectivity failure, token problem, or checker rejection. | Confirm token validity, worker selection, `.eflint` file, and task output. |
| Uploaded version does not appear in list | The upload targeted another worker, failed, or the list operation lacks connectivity. | Reinspect the selected worker context and task result before retrying. |
| Activation fails | Incorrect version identifier, expired/unauthorised token, checker unavailability, or rejected policy state. | List versions again, verify token status, and inspect the activation task output. |
| Workflow reaches the cluster but is denied | Deny-all is still active, the wrong version is active, or the active policy does not permit the requested operation. | Inspect active policy state and compare policy rules with actual workflow and data identifiers. |
| Workflow fails without a policy-denied result | Infrastructure, package, data, or workflow problem rather than policy logic. | Ask the User to inspect workflow-task status; ask the Administrator to run infrastructure health checks if services appear unavailable. |

## 8. Change record and hand-off

For each activation, record:

- target worker domain;
- policy source filename and source-control revision;
- uploaded policy version identifier;
- previous active version;
- activation time and responsible Policy Manager;
- reason for the change;
- expected verification workflow and outcome.

After activation, tell the User which policy condition has changed and which controlled workflow may be retried. If the workflow remains denied, compare the actual identifiers and requested data movement with the active policy before changing infrastructure.
