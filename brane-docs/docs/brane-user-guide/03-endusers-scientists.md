# Brane User Workflow Guide

**Audience:** Workflow users who discover resources, prepare workflows, and submit local or remote Brane runs.

This guide describes the user workflow supported by `brane-deployment` baseline `369392b991e0c3290739077d0ad071b5ce3f76bb`.

Users work with packages, datasets, configured instances, certificate bundles, workflows, and workflow results. Administrators deploy infrastructure and issue certificate bundles. Policy Managers upload and activate domain policies.

## User responsibilities

A user is responsible for:

1. discovering available packages and datasets;
2. configuring and selecting a Brane instance;
3. registering an Administrator-supplied certificate bundle;
4. building and testing packages where appropriate;
5. creating and submitting local or remote workflows;
6. monitoring workflow tasks and interpreting their outcome.

A user does not deploy infrastructure, create policy-manager tokens, upload policies, or activate policy versions.

## 1. Start from the deployment workspace

The `brane-deployment` repository provides user resources at its root:

```text
packages/
datasets/
certs/
workflow_codes/
```

The current baseline includes:

| Resource | Location | Purpose |
|---|---|---|
| Hello World package | `packages/hello_world/` | Minimal package and local-workflow example. |
| minmax package | `packages/minmax/` | Package example with associated dataset material. |
| minmax dataset configuration | `datasets/minmax/data/data.yml` | Dataset configuration source. |
| Certificate bundles | `certs/<domain>/` | Administrator-supplied domain access material. |

Check the local environment and available resources with the User helper:

```sh
cd /path/to/brane-deployment

bash scripts/brane_helper_user.sh
```

Choose **Check environment, connection & report**. The helper checks for the `brane` CLI, Docker, configured instances, and the repository-root `packages/`, `certs/`, and `datasets/` directories.

## 2. Discover packages and datasets

List built packages available to the local Brane CLI:

```sh
brane package list
```

List registered data resources visible through the current Brane configuration:

```sh
brane data list
```

Local source material can also be inspected directly:

```sh
cd /path/to/brane-deployment

find packages -maxdepth 2 -name container.yml -print
find datasets -maxdepth 3 -name data.yml -print
```

A source package under `packages/` is not automatically evidence that it has been built or is available to a remote workflow. Build and test it locally before relying on it.

## 3. Configure and select an instance

An Administrator supplies the central-node hostname or address and the intended instance name.

Add an instance and make it active:

```sh
brane instance add <CENTRAL_HOST> \
  --name <INSTANCE_NAME> \
  --use \
  --unchecked \
  --force
```

Inspect configured instances:

```sh
brane instance list
```

Select a configured instance before a remote submission:

```sh
brane instance select <INSTANCE_NAME>
```

The Streamlit equivalent is **User Workspace → Instances**. Use **Refresh configured instances** to inspect current configuration, then use **Add instance** when no suitable instance exists.

> **Note:** `--unchecked` bypasses validation while adding the instance. Confirm the supplied address with the Administrator before using it.

## 4. Register a certificate bundle

Remote execution requires the appropriate certificate bundle for the target domain. Request it from the Administrator; do not generate domain certificates yourself.

A complete local bundle contains:

```text
certs/<DOMAIN>/
├── ca.pem
├── client.pem
└── client-key.pem
```

Register it for the selected instance:

```sh
brane certs add \
  certs/<DOMAIN>/ca.pem \
  certs/<DOMAIN>/client.pem \
  certs/<DOMAIN>/client-key.pem \
  --instance <INSTANCE_NAME> \
  --domain <DOMAIN_HOST>
```

In the Streamlit interface, use **User Workspace → Certificates → Register certificate**. The interface checks that the selected bundle contains the required three files before starting the registration task.

Certificate private keys are secret material. Do not commit them, upload them to issue trackers, or paste their contents into task output.

## 5. Build and test a package

A package source directory contains a `container.yml` manifest. For example:

```text
packages/hello_world/container.yml
```

Build a package on Linux or another x86_64 build host:

```sh
brane package build --arch x86_64 packages/hello_world/container.yml
```

When building on Apple Silicon, the deployment nodes still require an x86_64 package target. Use the supplied wrapper:

```sh
cd /path/to/brane-deployment

bash scripts/package_build_macOS.sh packages/hello_world/container.yml
```

The wrapper checks for Docker and `brane`, configures an isolated Buildx builder if necessary, and invokes the x86_64 package build.

Test a built package locally:

```sh
brane package test hello_world
```

Use **User Workspace → Packages** for task-backed package builds and package-list inspection.

## 6. Create and run a workflow locally

A workflow submission needs:

- a workflow file ending in `.bs`;
- a user-supplied workflow label;
- packages referenced by that workflow.

Run a local workflow:

```sh
brane workflow run <WORKFLOW_USER> <WORKFLOW_FILE>
```

For the included Hello World example:

```sh
cd /path/to/brane-deployment

brane workflow run <WORKFLOW_USER> packages/hello_world/hello_world.bs
```

A user chooses their own workflow label. It identifies the submission; it is not an Administrator-provisioned login.

For a complete first local example, see [Hello, World!](../brane-tutorials/02-hello-world-example.md).

## 7. Submit a workflow remotely

Before remote submission, ensure that:

1. the intended instance is configured and selected;
2. the applicable certificate bundle is registered;
3. the target infrastructure is healthy;
4. the package and data references are correct;
5. an appropriate domain policy is active.

Submit through the selected instance:

```sh
brane instance select <INSTANCE_NAME>

brane workflow run --remote <WORKFLOW_USER> <WORKFLOW_FILE>
```

The shell helper provides an interactive version of the same sequence:

```sh
cd /path/to/brane-deployment

bash scripts/brane_helper_user.sh
```

Choose **Run workflow on remote domain**. It checks central connectivity, prompts for a workflow and user label, selects an instance, and then submits remotely.

The Streamlit equivalent is **Workflow Studio**:

1. enter a filename and BraneScript source;
2. enter a workflow label;
3. select **Remote configured instance**;
4. select the target instance;
5. select **Run workflow**.

Workflow Studio selects the requested instance immediately before remote submission.

## 8. Monitor tasks, results, and history

Frontend operations such as package builds, certificate registration, and workflow execution create persistent task records.

A task can be:

| Status | Meaning |
|---|---|
| `queued` | The task record exists and has not yet started. |
| `running` | The command is executing. |
| `succeeded` | The command exited with code `0`. |
| `failed` | The command exited with a non-zero code; inspect its log. |
| `interrupted` | The process is no longer present and its final exit state could not be recovered. |

After starting a task:

1. inspect the task monitor shown in the current workspace;
2. use **Refresh task status** while it is running;
3. open **Task History** for persistent logs and details;
4. filter Task History by role or status when needed.

The sidebar displays active tasks and recent task summaries. Task History stores task state and logs locally for the frontend; it is not a substitute for Administrator infrastructure health checks.

## 9. Interpret common user-facing failures

| Symptom | Likely cause | First action |
|---|---|---|
| `brane` command is unavailable | The CLI is not installed or not on `PATH`. | Use the workstation setup guidance or contact the Administrator. |
| Package build fails on Apple Silicon | The build target or Docker Buildx setup is unsuitable for x86_64 deployment nodes. | Use `scripts/package_build_macOS.sh` and inspect its task output. |
| No instance is available for remote execution | No instance has been configured. | Add the Administrator-supplied central address and instance name. |
| Certificate registration fails | The bundle is incomplete, the domain is wrong, or the certificate lacks required client extensions. | Check for `ca.pem`, `client.pem`, and `client-key.pem`; ask the Administrator to verify or regenerate the bundle. |
| Remote submission cannot connect | The selected instance, network path, or deployment infrastructure is unavailable. | Confirm the instance address; ask the Administrator to run the infrastructure health check. |
| Workflow is denied by policy | Deny-all remains active or the active policy does not permit the requested operation. | Contact the Policy Manager with the workflow, package, dataset, and domain identifiers. |
| Workflow task fails after submission | Workflow syntax, package inputs, dataset availability, package runtime, or infrastructure may be at fault. | Inspect Task History first; involve the Policy Manager for policy denial and the Administrator for infrastructure symptoms. |

## 10. Role hand-off

Use this escalation path:

```text
User: package, instance, certificate registration, workflow, task output
→ Policy Manager: policy permission, active version, policy denial
→ Administrator: certificates, instance endpoint, deployment health, infrastructure availability
```

A remote workflow can reach a healthy cluster and still be denied by policy. Treat this as a policy-management outcome unless task output indicates a connection or service failure.
