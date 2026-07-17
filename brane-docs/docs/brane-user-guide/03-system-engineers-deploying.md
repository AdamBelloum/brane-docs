## 3. System Engineers: Deploying Brane

This section describes how to set up a Brane instance (control node + worker nodes) and integrate it with your infrastructure.

### 3.1. Architecture: control node, worker nodes, proxies

Brane’s deployment model:

- **Control node (central Brane instance)**:
  - Runs:
    - Driver (receives workflows),
    - Planner (allocates tasks),
    - Central registry (packages, datasets, domains),
    - Central proxy (handles external access and certificates).

- **Worker nodes (domains/locations)**:
  - Run:
    - Delegate (connects to local compute backend),
    - Local registry (datasets & intermediate results),
    - Policy checker (PEP + reasoner),
    - Local proxy. 

- **Proxies and secure networking**:
  - Proxies expose services securely across domains.
  - Certificates (X.509) are used for mutual authentication. 

### 3.2. Installing dependencies

Before setting up nodes:

1. Install **Docker** on all relevant machines.
2. Install **BuildKit-based Docker Buildx** if your deployment uses it for building images. 
3. Install **Branectl** or equivalent management tooling:
   - Used to download/build images,
   - Generate configuration files,
   - Manage certificates.

### 3.3. Configuring the control node

Key configuration files:

- `infra.yml` – describes domains, locations, and registry layout.
- `node.yml` – describes node roles and services for the control node.
- `proxy.yml` – describes proxy configuration for secure external access.

Typical workflow:

1. Use Branectl (or similar tooling) to **generate initial configs** with sensible defaults.
2. Adjust `infra.yml` and `node.yml` to reflect your environment:
   - Domains and worker sites,
   - Hostnames and ports for services.
3. Configure `proxy.yml` to:
   - Expose required APIs,
   - Integrate with certificate and TLS setup.

### 3.4. Spinning up worker domains

For each worker domain:

1. Generate a `node.yml` describing:
   - Delegate,
   - Local registry,
   - Policy checker,
   - Local proxy. 

2. Configure the local compute backend:
   - Docker-based backend (simple),
   - Or a more advanced/custom backend (some older backends may be deprecated).

3. Start services:
   - Local registry,
   - Delegate,
   - Checker,
   - Proxy.

4. Register the domain with the control node’s `infra.yml` and central registry.

### 3.5. Certificates and secure networking

Brane uses X.509 certificates for mutual authentication: 

- **Server certificates**:
  - CA certificate (`ca.pem`),
  - Server certificates for control and worker nodes.

- **Client certificates**:
  - Identify users or services (e.g., scientists, admins).
  - Must be handled as identity proofs; if compromised, rights must be revoked.

Typical steps:

1. Generate CA and server certificates for each domain.
2. Distribute `ca.pem` appropriately (e.g., to the central node).
3. Generate and distribute client certificates to users:
   - Store them under config/certs,
   - Use them in the CLI and proxies.

Careful handling of certificates is essential for secure cross-domain communication.

---

*(Sections 4–8 will follow the same role-based structure and best-practice style. For now, we’ve set the skeleton and rewritten the core user-oriented parts. When you’re ready, we can start with Section 4 – Packages for software engineers, or Section 5 – Policies for policy experts, and then move on to workflows and admin topics.)*
