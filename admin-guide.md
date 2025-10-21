---
````markdown
# Administration & Operations Guide

_Last updated: October 2025_  
**Audience:** System engineers and infrastructure administrators  
**Purpose:** Describe how to manage Brane nodes, configure environments, and maintain distributed Brane systems.  
---

## 1. Understanding Node Types

Brane runs as a **distributed orchestration system**, composed of three types of nodes that collaborate securely.  

| **Node Type** | **Purpose** | **Main Components** | **Typical Host** |
|----------------|-------------|----------------------|------------------|
| **Control Node** | Central coordinator; manages workflows, package registry, and policies | `brane-api`, `brane-planner`, `brane-registry`, `brane-driver`, `aux-scylla` | Central server |
| **Worker Node** | Executes workflow tasks and packages | `brane-worker`, `brane-driver` | Compute node or cluster node |
| **Proxy Node** | Bridges isolated or cross-domain networks between sites | `brane-proxy` | Gateway or DMZ host |

Each node communicates securely using certificates and configuration files.  

---

## 2. Generating Configuration Files

Brane uses YAML configuration files to define infrastructure, nodes, backends, and proxies.  
All of these are generated using the **`branectl`** CLI.

> Use the `-f` flag to force overwriting existing files, and `-p <path>` to specify output directories.

---

###  `infra.yml`
Describes the overall infrastructure (which nodes exist and how they connect).

```bash
branectl generate infra -f -p ./config/infra.yml \
  amy:amy-worker-node.com bob:bob-worker-node.com
````

**Example output snippet:**

```yaml
locations:
  amy:
    host: amy-worker-node.com
  bob:
    host: bob-worker-node.com
```

---

### `node.yml`

Defines the configuration of a specific node (control, worker, or proxy).

#### Control node:

```bash
branectl generate node -f central central.example.com
```

#### Worker node:

```bash
branectl generate node -f worker central.example.com amy
```

**Example structure:**

```yaml
node:
  name: amy
  kind: worker
  control_plane: central.example.com
```

---

### `backend.yml`

Defines the compute backend for a worker node.
Usually “local” for on-machine Docker execution.

```bash
branectl generate backend -f -p ./config/backend.yml local
```

**Example:**

```yaml
backend:
  kind: local
```

---

### `proxy.yml`

Defines proxy configuration for cross-site communication.

```bash
branectl generate proxy -f -p ./config/proxy.yml
```

**Example:**

```yaml
proxy:
  host: proxy.example.com
  port: 50051
```

---

## 3. Managing Certificates

Each Brane node must have valid TLS certificates for secure communication.
Use `branectl` to generate and distribute them.

### Generate server certificates:

```bash
branectl generate certs -f -p ./config/certs server <LOCATION_ID> -H <HOSTNAME>
```

**Example:**

```bash
branectl generate certs -f -p ./config/certs server amy -H amy-worker-node.com
```

This command creates a directory:

```
config/certs/
 |--- ca.pem
 |--- cert.pem
 |--- key.pem
```

### Distribute certificates

* Copy each worker node’s CA certificate to the control node under:

  ```
  config/certs/<worker>/ca.pem
  ```
* Keep private keys secure. Do not share `key.pem`.

---

## 4. Managing Policies and Access Control

Brane integrates with **policy databases** to control data access, security, and computation boundaries.
These policies are typically expressed in **eFLINT** and managed locally per node.

### Generate a Policy Database

```bash
branectl generate policy_db -f -p ./config/policies.db
```

### Generate Policy Secrets (Deliberation and Expert)

```bash
branectl generate policy_secret -f -p ./config/policy_deliberation_secret.json
branectl generate policy_secret -f -p ./config/policy_expert_secret.json
```

### Generate Policy Token (For Authorization)

```bash
branectl generate policy_token -f -p ./config/policy_token.json
```

> These tokens authenticate and authorize actions performed by different roles in Brane’s policy framework.

---

##  5. Starting and Monitoring Nodes

After configuration, use `branectl start` to launch nodes.
All services run as Docker containers managed by Brane.

### Start Control Node

```bash
branectl start central
```

### Start Worker Node

```bash
branectl start worker
```

### Start Proxy Node

```bash
branectl start proxy
```

---

### Check Running Containers

```bash
docker ps
```

You should see containers such as:

```
brane-api
brane-planner
brane-registry
brane-driver
brane-proxy
aux-scylla
```

All should report **Status: Up**.

---

### Monitor Logs

View logs for a specific component:

```bash
docker logs brane-api
```

Tail logs live:

```bash
docker logs -f brane-api
```

---

### Stop or Restart Nodes

```bash
branectl stop central
branectl restart worker
```

---

## 6. Operational Verification

### Check Node Registration

```bash
brane instance list
```

Expected output example:

```
NAME        TYPE      STATUS
central     control   ACTIVE
amy         worker    ACTIVE
bob         worker    ACTIVE
```

### Verify Communication

On the control node:

```bash
brane ping amy
```

If successful, the control node can communicate securely with the worker node.

---

## 7. Maintenance Tips

* **Back up configuration files** (`infra.yml`, `node.yml`, and certs).
* **Rotate certificates** periodically using `branectl generate certs`.
* **Clean unused containers** to reclaim space:

  ```bash
  docker system prune -f
  ```
* **Keep Brane binaries updated**:

  ```bash
  branectl self-update
  ```

---

## Summary

| Task                  | Command                      | Purpose                        |
| --------------------- | ---------------------------- | ------------------------------ |
| Generate infra config | `branectl generate infra`    | Define overall topology        |
| Create node configs   | `branectl generate node`     | Configure control/worker/proxy |
| Generate certs        | `branectl generate certs`    | Secure node communication      |
| Manage policies       | `branectl generate policy_*` | Set access control and secrets |
| Start nodes           | `branectl start <type>`      | Run Brane components           |
| Check system          | `brane instance list`        | Verify active nodes            |

---

**With these operations, your Brane deployment is secure, connected, and ready for distributed workflows.**

Continue with the [Configuration & Reference Manual ?] (reference-manual.md)

[? Back to Table of Contents](Brane-documentation-index.md)

```
