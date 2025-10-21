---
````markdown
# Brane Installation & Setup Guide

_Last updated: October 2025_  
**Audience:** System administrators, developers testing locally  
**Purpose:** Explain how to install Brane for both local (single node) and distributed (multi-node) environments.  
---

## 1. Prerequisites

Before installing Brane, ensure your system meets the following requirements.

### Runtime Dependencies

| Component | Description | Installation Notes |
|------------|--------------|--------------------|
| **Docker** | Runs Brane services as containers | [Docker Install Docs](https://docs.docker.com/get-docker/) |
| **Docker BuildKit** | Builds container images efficiently | Included in modern Docker releases; enable with `docker buildx create --use` |
| **OpenSSL** | Required by Brane command-line tools | macOS: `brew install openssl`<br>Ubuntu/Debian: `sudo apt install openssl` |
| **GLIBC ≥ 2.27** | Needed for precompiled binaries | Check with `ldd --version` |

> Tip: On Linux, add your user to the Docker group to avoid using `sudo`:
> ```bash
> sudo usermod -aG docker "$USER"
> ```
> Then log out and log back in.

---

## 2. Installing `branectl`

`branectl` is the **administration CLI** used to install, configure, and manage Brane nodes.

###  macOS (Intel)
```bash
sudo wget -O /usr/local/bin/branectl \
  https://github.com/BraneFramework/brane/releases/download/nightly/brane-macos-x86_64
````


###  macOS (Apple Silicon)

```bash
sudo wget -O /usr/local/bin/branectl \
  https://github.com/BraneFramework/brane/releases/download/nightly/brane-macos-aarch64
```

###  Linux (x86-64)

```bash
sudo wget -O /usr/local/bin/branectl \
  https://github.com/BraneFramework/brane/releases/download/nightly/brane-linux-x86_64
```

Make it executable:

```bash
sudo chmod +x /usr/local/bin/branectl
```

Verify installation:

```bash
branectl --help
```

If you see usage instructions, the tool is installed correctly.

---

## 3. Installing Brane on a Single Node (Local Test)

A local installation is ideal for learning, prototyping, or testing workflows.

### Step 1 - Download Brane Service Images

```bash
branectl download services central -f
branectl download services auxillary -f
```

### Step 2 - Generate Default Configurations

```bash
branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f central localhost
```

This creates configuration files under `./config/` and `node.yml` in the current directory.

### Step 3 - Start the Local Instance

```bash
branectl start central
```

> Note: The ScyllaDB database container may take up to a minute to become fully available.
> Use `watch docker ps` to monitor the container status.

Once all containers (including `brane-api`) are running, Brane is ready for use.

---

## 4. Installing a Multi-Node Brane Cluster

A distributed setup typically includes:

* **Control Node** - orchestrates workflows
* **Worker Nodes** - execute computations
* **Proxy Nodes** *(optional)* - handle cross-site network communication

---

### Step 1 - Prepare Each Machine

Install the prerequisites and `branectl` on all machines that will host Brane nodes.

---

### Step 2 - Create Certificates for Secure Communication

On each node:

```bash
branectl generate certs -f -p ./config/certs server <LOCATION_ID> -H <HOSTNAME>
```

Example:

```bash
branectl generate certs -f -p ./config/certs server amy -H amy-worker-node.com
```

Copy each node’s `ca.pem` file to the **control node** under `config/certs/<node>/ca.pem`.

---

### Step 3 - Set Up the Control Node

Generate the infrastructure and proxy configuration:

```bash
branectl generate infra -f -p ./config/infra.yml amy:amy-worker-node.com bob:192.0.2.2
branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f central central.example.com
```

Start the control node:

```bash
branectl start central
```

---

### Step 4 - Set Up Each Worker Node

On each worker node:

```bash
branectl download services worker -f
branectl generate backend -f -p ./config/backend.yml local
branectl generate proxy -f -p ./config/proxy.yml
branectl generate policy_secret -f -p ./config/policy_deliberation_secret.json
branectl generate policy_secret -f -p ./config/policy_expert_secret.json
branectl generate policy_db -f -p ./policies.db
branectl generate node -f worker <CENTRAL_HOSTNAME> <LOCATION_ID>
```

Example:

```bash
branectl generate node -f worker central.example.com bob
```

Start the worker node:

```bash
branectl start worker
```

---

### Step 5 - Optional: Deploy a Proxy Node

If needed for cross-domain communication:

```bash
branectl download services proxy -f
branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f proxy proxy.example.com
branectl start proxy
```

---

## 5. Verifying the Deployment

### Check Running Containers

```bash
docker ps
```

You should see containers for:

* `brane-api`
* `brane-driver`
* `brane-planner`
* `brane-registry`
* `brane-proxy`
* `aux-scylla` (database)

All should have a **Status: Up**.

### Check Logs

To view logs of a specific service:

```bash
docker logs <container_name>
```

Example:

```bash
docker logs brane-api
```

### Verify CLI Connection

Run:

```bash
brane instance list
```

If your nodes appear, the setup is complete.

---

## Summary

| Step | Action               | Outcome                              |
| ---- | -------------------- | ------------------------------------ |
| 1    | Install dependencies | Docker and OpenSSL installed         |
| 2    | Install `branectl`   | CLI verified with `branectl --help`  |
| 3    | Run local instance   | Single-node Brane up and running     |
| 4    | Configure multi-node | Control, worker, and proxy connected |
| 5    | Verify setup         | Containers active and reachable      |

---

Brane is now installed and operational.
You can proceed to [Build Your First Package ?](developer-guide.md) or
[Run Your First Workflow ?](getting-started.md)
or [? Back to Table of Contents](Brane-documentation-index.md)


```
---

```
