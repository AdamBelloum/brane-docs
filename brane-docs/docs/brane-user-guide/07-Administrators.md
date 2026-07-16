## 7. Administrators: Instances, Credentials, and Multi-Domain Use

This section is for **Brane administrators** and anyone responsible for:

- Managing Brane instances and registries,
- Handling client credentials and certificates,
- Supporting interactions with multiple domains and remote instances. [1]

It assumes Brane is already deployed (see Section 3) and focuses on day-to-day administration and secure multi-domain usage.

---

### 7.1. Managing instances and registries

A **Brane instance** is the combination of:

- A central control node (driver, planner, central registry, central proxy),
- One or more worker domains (delegates, local registries, checkers, local proxies). [1]

Administrators manage:

- Which instances the CLI knows about,
- How the CLI authenticates to each instance,
- How packages and datasets are registered and discovered.

#### 7.1.1. Instance configuration (client-side)

On the **client machine**, the Brane CLI maintains configuration describing:

- Known instances (e.g., URLs / hostnames of central proxies),
- Credentials for each instance (certificates, tokens),
- Optional per-instance settings (e.g., default domain). [1]

Conceptually (syntax to be validated):

- You add or configure instances using CLI commands like:
  - `brane instance add` / `brane instance config`
  - `brane instance list` to see known instances.
- Each instance entry includes:
  - An endpoint (URL/host),
  - Reference to certificates or tokens used to authenticate.

Exact command names and options will be updated later according to current CLI versions.

#### 7.1.2. Central and local registries

Registries track:

- **Packages** (ECUs),
- **Datasets** and intermediate results,
- **Domains** participating in the deployment. [1]

Administrators:

- Ensure the **central registry**:
  - Knows about all relevant domains,
  - Is reachable from worker nodes and clients.
- Ensure **local registries** on worker domains:
  - Track datasets and intermediate results correctly,
  - Synchronize with central registry when needed. [1]

Typical tasks:

- Check registry health and connectivity (e.g., via registry APIs).
- Verify that new domains appear correctly in the central registry after configuration changes.
- Ensure package publishing (see Section 4) results in visible entries for scientists.

---

### 7.2. Client certificates and identities

Brane uses **X.509 certificates** for mutual authentication and identity:

- **Server certificates** authenticate Brane services.
- **Client certificates** authenticate users and services connecting to the instance. [1]

Administrators manage:

- CA and server certificates for each domain,
- Client certificates for users (scientists, developers, admins),
- Revocation or removal of compromised credentials.

#### 7.2.1. Certificate layout

The manual describes a typical layout: [1]

- A `config/certs` directory on the client and/or server, containing:
  - CA certificates (`ca.pem`),
  - Server certificates and keys,
  - Client certificates and keys (per user or service).

Each **domain** may have its own certificate directory, and the central node must know the CA certificates of domains it trusts.

#### 7.2.2. Generating certificates

Branectl (or associated tooling) can generate:

- CA certificates,
- Server certificates,
- Client certificates. [1]

Conceptually (commands to be validated):

- You run a tool like `branectl cert generate` with:
  - Domain name or node role (control, worker, proxy),
  - Output directories for certs.

The generated artifacts include:

- `ca.pem` (CA certificate),
- `server.pem` / `server-key.pem`,
- `client.pem` / `client-key.pem` for each user or service.

The **CA certificate** must often be shared with the central node so it can verify domain/server certificates.

#### 7.2.3. Distributing and protecting client certificates

Client certificates are **identity proofs**: [1]

- Each scientist, developer, or admin may have a unique certificate.
- Certificates must be stored securely (e.g., protected file permissions, secure distribution).

Administrators should:

- Distribute client certificates to users via secure channels.
- Avoid sharing keys via unprotected media (e.g., email attachments without encryption).
- Document procedures for revoking access if a certificate is compromised:
  - Remove or disable certificate on the instance,
  - Clean up local config or credential entries.

On the client side, Brane CLI configuration points to:

- The correct certificate and key files for each instance.
- The CA certificate used to verify the instance.

---

### 7.3. Working with multiple domains and instances

Brane supports interaction with **multiple domains** and even multiple instances. [1]

Typical scenarios:

- A scientist’s CLI interacts with:
  - A main Brane instance,
  - One or more external domains with their own registries.
- Packages and datasets may be hosted on different domains.

Administrators help users:

- Configure the CLI for multiple instances.
- Install appropriate CA and client certificates.
- Understand which instance they are targeting with each command.

#### 7.3.1. Multiple instances for one client

On a single client machine, you may have:

- Several instance entries (e.g., `prod`, `test`, `partner-domain`).
- Different credentials for each instance. [1]

Conceptually:

- Use CLI commands like:
  - `brane instance list` to view instances,
  - `brane instance use <name>` to select an active instance,
  - Instance-specific options for commands (e.g., flags or environment variables).

Administrators should:

- Provide clear naming and documentation for instances.
- Ensure users know which instance hosts which packages/datasets.
- Use consistent configuration conventions (directories, file names).

#### 7.3.2. Cross-domain data access

Cross-domain access involves:

- Brane instance contacting worker domains,
- Worker registries and delegates interacting with other domains,
- Policy checker enforcing rules. [1]

Administrators must:

- Ensure necessary CA/server certificates are installed for cross-domain trust.
- Coordinate with policy experts (Section 5) to define policies governing:
  - Which datasets may be accessed from which domains,
  - Under what conditions transfers are allowed.

---

### 7.4. Cleaning up credentials and revoking access

Credential and identity management must include **revocation and cleanup** procedures.

#### 7.4.1. Removing local credentials

On the client side, if a user changes roles or loses access:

- Remove or archive:
  - Client certificates and keys,
  - Instance configuration entries. [1]

Conceptually:

- Use CLI commands to:
  - Remove instance entries (e.g., `brane instance remove <name>`),
  - Clear stored credentials.
- Manually delete certificate files if needed, following security policies.

#### 7.4.2. Revoking server-side access

On the server side, if a certificate is compromised or a user leaves:

- Administrators must:
  - Remove or disable the user’s certificate from trust stores,
  - Update policy or access control lists if they use certificate-based rules. [1]

Depending on the deployment, this may involve:

- Updating proxy or gateway configurations,
- Adjusting policy checker rules,
- Restarting services to apply changes.

#### 7.4.3. Best practices

Recommended practices for administrators:

- Maintain a registry of issued client certificates (who, when, which instance).
- Use short-lived certificates or rotate them periodically.
- Document revocation procedures and test them regularly.
- Coordinate with system engineers and policy experts to ensure that certificate changes do not silently break workflows.

---

### 7.5. Summary

For administrators, Brane management focuses on:

- **Instances and registries** – knowing which instances exist, how they are configured, and how packages/datasets are tracked.
- **Certificates and identities** – issuing, distributing, and revoking client and server certificates securely.
- **Multi-domain usage** – configuring clients and instances to interact across domains with clear trust boundaries.
- **Cleanup and revocation** – ensuring compromised or outdated credentials are removed and access is correctly revoked.

These tasks complement the infrastructure work of system engineers (Section 3), the package/development work of software engineers (Section 4), and the policy responsibilities of data policy experts (Section 5). [1]