# Brane Infrastructure Deployment & Administration Guide

**Audience:** System Administrators, Infrastructure Engineers  
**Purpose:** Architecture overview, multi-node deployment, certificate management, and day-to-day administration of the Brane ecosystem.

---

## 1. Architectural Overview & Concepts

Before deploying the infrastructure, it is critical to understand how Brane instances, domains, and registries interact.

### 1.1 The Brane Instance
A complete Brane instance consists of a central orchestrator connected to decentralized computational areas:

*   **Central Control Node:** Runs core orchestration services including the driver, planner, central registry, and central proxy.

*   **Worker Domains:** Distinct infrastructure sites hosting delegates, local registries, policy checkers, and local proxies where actual computation takes place.

### 1.2 Registry Architecture

*   **Central Registry:** Hosted on the Control Node. Tracks all executable packages (ECUs), datasets, and known participating domains. It must be accessible by both worker nodes and client CLIs.

*   **Local Registries:** Hosted within individual worker domains. They track local datasets and intermediate results, synchronizing with the central registry when necessary.

---

## 2. Prerequisites & Infrastructure Setup

We assume you have provisioned your physical or virtual hosts and configured network routing between them. Ensure every machine meets the following baseline requirements.

### Runtime Dependencies

| Component | Description | Installation Notes |
|------------|--------------|--------------------|
| **Docker** | Runs Brane services as containers | [Docker Install Docs](https://docs.docker.com/get-docker/) |
| **Docker BuildKit** | Builds container images efficiently | Included in modern Docker releases; enable with `docker buildx create --use` |
| **OpenSSL** | Required for cryptographic operations | macOS: `brew install openssl`<br>Ubuntu/Debian: `sudo apt install openssl` |
| **GLIBC $\ge$ 2.27** | Needed for precompiled binaries | Check with `ldd --version` |

> **Tip:** To avoid running infrastructure commands as root, add your deployment user to the Docker group:
> ```bash
> sudo usermod -aG docker "$USER"
> ```
> Log out and back in for changes to take effect.


---

## 0. Installation & Setup
 
Before building packages, you must download the pre-compiled `branectl` binary directly from the official GitHub releases page and add it to your system path.
see instruction in the guide [installing brane tools](08-brane-tools.md)

---

> **Prefer Automation?**
> If you havei:
>
>  - SSH access to your target hosts, 
>
>  - known IP addresses/hostnames, 
>
>  - and a correctly configured gRPC network, 
>
>  - you can skip the manual commands below.
> 
> The is an  Ansible playbook that automates the entire infrastructure deployment (including dependency installation, certificate generation, and service orchestration).
>
>  - More details in [Brane Deployment Repository (brane-deployment)](https://github.com/AdamBelloum/brane-deployment).

---

## 3. Public Key Infrastructure (PKI) & Certificates

Brane enforces mutual authentication (mTLS) via **X.509 certificates** to guarantee identity and secure cross-domain data access.

### 3.1 Directory Layout

Every node and client utilizes a standardized `config/certs` directory layout:

* `ca.pem`: The Certificate Authority certificate used to establish trust.


* `server.pem` / `server-key.pem`: Authenticates Brane services.

* `client.pem` / `client-key.pem`: Authenticates developers, scientists, or automated systems connecting to the instance.

### 3.2 Generating Certificates

Execute the following on your target nodes to generate the required cryptographic assets:

```bash
branectl generate certs -f -p ./config/certs server <LOCATION_ID> -H <NODE_HOSTNAME_OR_IP>

```

> **Critical Security Step:** Securely copy each worker node’s `ca.pem` file back to the **control node** under `config/certs/<LOCATION_ID>/ca.pem` so the central orchestrator trusts the incoming worker connections.
> 
> 

---

## 4. Understanding Document Placeholders

To get your cluster running as quickly as possible, map the abstract placeholders in the deployment section to your actual infrastructure values:

* **`<LOCATION_ID>` / `<WORKER_ID>**`: A friendly alphanumeric label you invent to name a worker node (e.g., `amsterdam`, `worker-prod-1`). Once you choose a label for a node, use it consistently everywhere.
* **`<CENTRAL_HOSTNAME_OR_IP>`**: The static network address (internal IP, public IP, or internal FQDN) that worker nodes and clients use to connect to your central management host.

### Quick-Start Network Blueprint (Concrete Example)

Throughout Section 5, we assume a network layout of three provisioned hosts:

* **Central VM IP:** `10.0.0.10`
* **Worker A VM IP:** `10.0.0.20` (We choose the location ID string: `amsterdam`)
* **Worker B VM IP:** `10.0.0.30` (We choose the location ID string: `berlin`)

---

## 5. Deployment Topologies

### Topology A: Single-Node Deployment (Local Test/Prototyping)

Ideal for local validation or staging environments before distributed rollout.

```bash
# 1. Download Core Images
branectl download services central -f
branectl download services auxillary -f

# 2. Generate Local Configs
branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f central localhost

# 3. Spin up Instance
branectl start central

```

Note: The `aux-scylla` database container can take up to 60 seconds to fully initialize. Use `watch docker ps` to verify readiness.

---

### Topology B: Multi-Node Distributed Cluster (Production)

#### Step 1: Control Node Setup

On the central orchestrator machine (`10.0.0.10`), define your cluster layout by mapping worker location IDs to their network addresses:

```bash
# Template Syntax:
# branectl generate infra -f -p ./config/infra.yml <WORKER_ID_1>:<WORKER_IP_OR_HOST_1> <WORKER_ID_2>:<WORKER_IP_OR_HOST_2>

# Real-World Blueprint Execution:
branectl generate infra -f -p ./config/infra.yml amsterdam:10.0.0.20 berlin:10.0.0.30

branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f central 10.0.0.10

# Start central services
branectl start central

```

#### Step 2: Worker Domain Setup

On each provisioned worker host, configure local backends, proxies, and security policies:

```bash
# Download worker binaries
branectl download services worker -f

# Generate policies, storage backends, and token secrets
branectl generate backend -f -p ./config/backend.yml local
branectl generate proxy -f -p ./config/proxy.yml
branectl generate policy_secret -f -p ./config/policy_deliberation_secret.json
branectl generate policy_secret -f -p ./config/policy_expert_secret.json
branectl generate policy_db -f -p ./policies.db

# Template Syntax:
# branectl generate node -f worker <CENTRAL_HOSTNAME_OR_IP> <LOCAL_WORKER_ID>

# Real-World Blueprint Execution (Run this on Worker host 10.0.0.20):
branectl generate node -f worker 10.0.0.10 amsterdam

# Real-World Blueprint Execution (Run this on Worker host 10.0.0.30):
# branectl generate node -f worker 10.0.0.10 berlin

branectl start worker

```

#### Step 3: Optional Isolated Proxy Node

If cross-site communication is blocked by aggressive firewalls, spin up dedicated proxy gateways:

```bash
branectl download services proxy -f
branectl generate proxy -f -p ./config/proxy.yml
branectl generate node -f proxy <PROXY_HOSTNAME_OR_IP>
branectl start proxy

```

---

## 6. Deployment Verification & Troubleshooting

### 6.1 Container Status Check

Run `docker ps` on your hosts. You should observe active containers with a status of **Up**:

* `brane-api` & `brane-driver` (Control node only)

* `brane-planner` & `brane-registry` (Control node only)

* `aux-scylla` (Central database cluster)

* `brane-proxy` (Present on all nodes handling transit)

### 6.2 Reviewing System Logs

To troubleshoot startup faults or connection rejections, stream the application logs directly from Docker:

```bash
docker logs -f brane-api
docker logs -f brane-proxy
```
---

## 7. Day-to-Day User Provisioning & Administration

Once the cluster infrastructure is verified, the administrator handles user access management, multi-domain routing, and credential rollouts.

### 7.1 Onboarding Users (Client-Side Setup)

End-users (data scientists, developers) use the standard `brane` user CLI. To grant access to an instance, distribute their unique **Client Certificates** via secure internal channels.

The user runs the following commands on their machine to pair with your deployed infrastructure:

```bash
# View current cluster configurations
brane instance list

# Template Syntax:
# brane instance add <INSTANCE_NAME> <CENTRAL_PROXY_URL> --cert ./config/certs/client.pem --key ./config/certs/client-key.pem

# Real-World Blueprint Execution:
brane instance add cluster-prod 10.0.0.10 --cert ./config/certs/client.pem --key ./config/certs/client-key.pem

# Select instance as default target
brane instance use cluster-prod
```

### 7.2 Managing Access & Certificate Revocation

If a client certificate is compromised, or an employee changes roles, security must be updated server-side:

1. **Server-Side Revocation:** Remove or disable the certificate from the trust stores/volumes mapped to your running `brane-proxy` or policy checker configurations.

2. **Restart Security Services:** To instantly force clear connection caches:
```bash
docker restart brane-proxy

```

3. **Client-Side Cleanup:** Instruct the user to drop the configuration records:

```bash
brane instance remove cluster-prod

```
---

## Summary Lifecycle Checksheet

| Phase | Core Administrative Action | Target Validation |
| --- | --- | --- |
| **1. Prep** | Meet system requirements & install `branectl`.  | `branectl --help` works cleanly.  |
| **2. Securing** | Generate CA, server, and client certificates.  | Files populate `config/certs/`.  |
| **3. Deploy** | Execute `branectl start` for target nodes.  | All core containers show **Status: Up**.  |
| **4. Onboard** | Deliver client certs; link user CLIs.  | User runs `brane instance list` successfully.  |
| **5. Rotate** | Update policies and revoke dead credentials.  | Compromised endpoints return authentication errors.  |

```

```
