# Brane Administrator Operations Guide

**Audience:** Brane infrastructure administrators.

This guide describes the supported operational workflow for the Docker/Ansible deployment in `brane-deployment` at baseline revision `369392b991e0c3290739077d0ad071b5ce3f76bb`.

Administrators deploy and maintain infrastructure, verify its health, and provision domain access material. Package testing and workflow submission belong to the User workflow. Policy upload and activation belong to the Policy Manager workflow.

## Administrator responsibilities

Administrators are responsible for:

1. maintaining the Ansible inventory and deployment prerequisites;
2. deploying, redeploying, and recovering the central and worker infrastructure;
3. running infrastructure health checks using administrator SSH access;
4. generating and securely distributing client-certificate bundles;
5. generating and securely distributing policy-manager tokens;
6. using destructive cleanup only when a complete redeployment is intended.

The Streamlit **Administration** workspace exposes the same four primary operational areas:

- Deploy infrastructure;
- Run health check;
- Domain certificates;
- Policy-manager tokens.

Long-running frontend actions run as monitored tasks. Do not place certificate private keys or token values in browser-visible logs.

## 1. Prerequisites and inventory

Run administration commands from the `brane-deployment` control-machine checkout.

The deployment requires:

- a Python environment containing Ansible;
- SSH access from the control machine to the central node and all worker nodes;
- an Ansible inventory defining the `central` and `workers` groups;
- Docker available on deployment nodes;
- a deployment user and host addresses configured in the inventory;
- the required deployment variables, including central and worker installation directories.

The active inventory is normally:

```text
docker-deployment/inventories/production/hosts.ini
```

This inventory is deployment-specific and may contain sensitive host information. Keep it outside published documentation and do not commit credentials or private keys.

Before making infrastructure changes, review the planned execution:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
ansible-playbook -i inventories/production/hosts.ini site.yml \
  --syntax-check

PATH="../venv/bin:$PATH" \
ansible-playbook -i inventories/production/hosts.ini site.yml \
  --check --diff
```

`--check --diff` is a planning aid. It does not prove that all deployment actions can run successfully.

## 2. Deploy or redeploy infrastructure

The supported deployment entry point is the Ansible playbook:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
ansible-playbook -i inventories/production/hosts.ini site.yml
```

A full deployment runs all configured phases. Use it for a fresh deployment or when the intended change spans several phases.

### Deployment phases

| Tag | Purpose |
|---|---|
| `prerequisites` | Install or configure required node prerequisites. |
| `branectl` | Install `branectl` on inventory nodes. |
| `branecli` | Install `brane` on inventory nodes. |
| `workers` | Configure and prepare worker nodes. |
| `central` | Configure and prepare the central node. |
| `certs` | Exchange node certificate authority material between central and workers. |
| `start` | Start Brane services. |
| `smoke` | Run the deployment smoke test from the central node. |

For a controlled partial deployment, select only the required tag:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
ansible-playbook -i inventories/production/hosts.ini site.yml \
  --tags workers
```

For a fresh deployment, the normal dependency order is:

```text
prerequisites → branectl → branecli → workers → central → certs → start → smoke
```

Do not use superseded manual lifecycle procedures from older Brane distributions as deployment instructions for this environment.

## 3. Verify infrastructure health

Infrastructure health checks require administrator SSH access to the central node and workers. They read the Ansible inventory, inspect the deployment, and return:

- exit code `0` when all checks pass;
- exit code `1` when one or more checks fail.

Run the complete report after deployment, redeployment, certificate exchange, or a suspected infrastructure failure:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
bash ../scripts/brane_healthcheck.sh --report
```

To use a non-default inventory:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
bash ../scripts/brane_healthcheck.sh \
  --inventory /path/to/hosts.ini \
  --report
```

To narrow the check to one inventory node:

```sh
cd /path/to/brane-deployment/docker-deployment

PATH="../venv/bin:$PATH" \
bash ../scripts/brane_healthcheck.sh \
  --node <INVENTORY_NODE> \
  --report
```

The health check verifies deployment resources such as required containers, listening ports, and Docker mounts. Treat a failed health check as an infrastructure issue before troubleshooting a user workflow.

## 4. Generate and distribute client certificates

Generate a client-certificate bundle for a selected central or worker node/domain:

```sh
cd /path/to/brane-deployment

PATH="venv/bin:$PATH" \
bash scripts/brane_gen_cert.sh \
  --inventory docker-deployment/inventories/production/hosts.ini \
  --node <INVENTORY_NODE> \
  --output-name <DOMAIN_LABEL>
```

The helper:

1. reads the selected node from the Ansible inventory;
2. verifies that the remote node has its certificate authority material;
3. creates and signs a client certificate on that node;
4. replaces that node's current `client.pem` and `client-key.pem`;
5. copies `ca.pem`, `client.pem`, and `client-key.pem` to:

```text
certs/<DOMAIN_LABEL>/
```

> **Warning:** generation replaces the selected node/domain's existing client certificate and private key. Confirm the intended node and maintain any required certificate-rotation record before proceeding.

The generated bundle contains private key material. Transfer it only through an approved secure channel. Do not attach it to public issue trackers, commit it to Git, print it in logs, or paste it into shell-history examples.

A user registers the supplied bundle for a selected Brane instance with:

```sh
brane certs add <CA_CERT> <CLIENT_CERT> <CLIENT_KEY> \
  --instance <INSTANCE_NAME> \
  --domain <DOMAIN_HOST>
```

The administrator provides the correct domain hostname and bundle; the user performs instance-level registration as part of their own workflow.

## 5. Generate and distribute policy-manager tokens

A policy-manager token authorises policy operations for a specific domain. Generate it only for an identified policy manager, domain, and validity period.

### Preferred method: Administration workspace

In the Streamlit **Administration** workspace:

1. Open **Policy-manager tokens**.
2. Enter the policy manager name.
3. Select the target domain.
4. Enter the required validity period.
5. Generate the token.
6. Download the generated token file and transfer it securely.

The implementation stores generated token files with owner-only permissions and does not display token values in task logs. The policy manager should receive the file through an approved secure channel.

### Shell-helper method

The administrator helper provides an interactive token-generation action:

```sh
cd /path/to/brane-deployment

bash scripts/brane_helper_admin.sh
```

Choose **Generate policy expert token for policy manager** and provide:

- the policy manager name;
- the worker hostname or domain identifier;
- a validity period, such as `30d`;
- a secure output path.

The helper invokes `branectl generate policy_token` with the administrator-controlled policy secret. Do not copy the secret path, secret content, or generated JWT into documentation or terminal-history examples.

## 6. Full destructive cleanup

`scripts/brane_cleanup.sh` is a recovery operation for a complete reset. It requires interactive confirmation by typing `YES`.

It removes, on every inventory node:

- Brane containers;
- Brane images and Docker volumes;
- Brane deployment directories;
- generated deployment configuration, certificates, and policy data inside those directories;
- selected Brane release archives from temporary storage.

It preserves:

- SSH keys and `authorized_keys`;
- non-Brane Docker resources;
- Docker itself and installed system packages;
- files outside the Brane deployment directories.

Run cleanup only when you intend to remove the deployment and perform a full redeployment afterward:

```sh
cd /path/to/brane-deployment

PATH="venv/bin:$PATH" \
bash scripts/brane_cleanup.sh \
  --inventory docker-deployment/inventories/production/hosts.ini
```

After cleanup, redeploy using the full Ansible workflow in [Deploy or redeploy infrastructure](#2-deploy-or-redeploy-infrastructure). Do not use cleanup as a routine troubleshooting step.

## 7. Operational hand-off

After a healthy deployment:

1. The **Administrator** confirms infrastructure health and securely provisions certificates or policy-manager tokens.
2. The **Policy Manager** uploads and activates the applicable domain policy.
3. The **User** builds or discovers resources, registers their certificate bundle, and submits workflows.

The default deployment policy is deny-all. A remote workflow may reach the cluster successfully but still be denied until the appropriate policy is active.

## 8. Troubleshooting boundaries

| Symptom | First responsible role | First action |
|---|---|---|
| Health check fails | Administrator | Inspect the full health report, inventory connectivity, containers, ports, and mounts. |
| Certificate generation fails | Administrator | Check SSH access, inventory node selection, and certificate authority material on the target node. |
| User cannot register or use a certificate | Administrator and User | Verify the supplied bundle, selected instance, and target domain. |
| Policy upload or activation fails | Policy Manager | Check the policy workflow, token validity, policy input, and checker connectivity. |
| Workflow is denied by policy | Policy Manager | Confirm the active policy permits the requested workflow and data movement. |
| Workflow execution fails after policy approval | User, then Administrator | Check workflow inputs and task status; use the infrastructure health report if the failure indicates service unavailability. |

For deployed-service topology and ports, see the [Deployed Brane Architecture Reference](../brane-spec/02-architecture.md).
