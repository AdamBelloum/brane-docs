# 4. Data and Configuration Model

This section describes the configuration artifacts that define how Brane sees:

- Packages (executable code units),
- Datasets,
- Infrastructure and node layout.

Brane uses the following primary configuration files:

- `container.yml` – package (ECU) specification  
- `data.yml` – dataset specification  
- `infra.yml` – infrastructure and domain layout  
- `node.yml` – per-node service and network configuration

## 4.1. Package specification (`container.yml`)

### 4.1.1. Purpose

`container.yml` describes a **Brane package**, i.e. an Executable Code Unit (ECU) that Brane can call from workflows. It defines:

- Metadata (name, version, description, owner),
- Runtime environment (base image, dependencies, environment variables),
- Files to include,
- The **entrypoint** and functions (actions) exposed to workflows.

### 4.1.2. Required fields

At minimum, a valid ECU must define:

- `name`  
  Unique identifier of the package.

- `version`  
  Version string.

- `kind`  
  Must be `ecu` for executable code units.

- `entrypoint`  
  How Brane runs the package (command or script invoked by workers).

These fields allow Brane to register, resolve, and invoke the package.

### 4.1.3. Optional metadata

Optional metadata includes:

- `owners`  
  Owners or maintainers of the package.

- `description`  
  Human-readable description of what the package does.

Metadata supports discovery and provenance.

### 4.1.4. Build and runtime environment

To make ECUs reproducible and portable, `container.yml` describes the environment:

- `base` / `image`  
  Container image used as the base.

- `env`  
  Environment variables inside the container.

- `dependencies`  
  System-level dependencies (e.g., apt packages).

- `install`  
  Steps to prepare the ECU (install libraries, build binaries).

- `files`  
  Files to copy into the container.

- `postinstall`  
  Steps after installation and copying, often validations or final configuration.

Workers use this description to build and run ECUs consistently.

### 4.1.5. Exposed functions / actions

Packages expose functions (actions) that workflows can call. These are:

- Referenced in WIR graphs,
- Executed by workers according to the entrypoint and arguments.

## 4.2. Dataset specification (`data.yml`)

### 4.2.1. Purpose and basic model

`data.yml` describes **datasets** that workflows can use. Brane distinguishes:

- **Data** – persistent datasets,
- **IntermediateResult** – results produced during workflow execution.

Both are tracked by names (DataNames).

### 4.2.2. Access kinds

Datasets are accessed through an **access mechanism**:

- `AccessKind::File`  
  Dataset is accessible as files. Configuration specifies:
  - Path(s) on the domain’s file system,
  - Parameters for file access (e.g., read-only).

Other access kinds can be added as the system evolves.

### 4.2.3. Ownership and description

Datasets can carry metadata:

- `owners`  
  Responsible domain or individuals.

- `description`  
  Explanation of contents and intended use.

This supports governance and documentation.

### 4.2.4. Transfers and preprocessing

When a workflow needs data that is not locally available, Brane uses **preprocessing kinds**:

- `PreprocessKind::TransferRegistryTar`  
  Describes transfer via registry tar archives:
  - Source domain or registry,
  - Location of the archive,
  - Address/mechanism for transfer.

Preprocess kinds express how datasets and intermediate results are moved between domains.

## 4.3. Infrastructure configuration (`infra.yml`, `node.yml`)

Brane separates **logical infrastructure** from **node-level service configuration**.

### 4.3.1. `infra.yml` – domains and locations

`infra.yml` describes:

- Domains and locations,
- Delegates/registries for worker nodes.

The orchestrator uses `infra.yml` to maintain a distributed view of domains, workers, and registries.

### 4.3.2. `node.yml` – node roles and service maps

`node.yml` defines:

- Hostnames and network mappings per node,
- Roles and services provided by each node.

Typical node roles:

- Central nodes (orchestrator services, global registries),
- Worker nodes (task execution),
- Proxy nodes (communication and exposure).

Service maps describe which services are exposed and on which interfaces.

### 4.3.3. Public, private, and external interfaces

Services can be exposed via:

- **Public**  
  Reachable by other domains or the orchestrator (e.g., `api/registry`, `drv/driver`, `prx/proxy`).

- **Private**  
  Restricted to local node or internal domain network.

- **External**  
  Exposed to environments outside the Brane federation (user-facing APIs, integrations).

## 4.4. Summary of configuration responsibilities

- `container.yml`  
  Defines how code is packaged and executed as ECUs.

- `data.yml`  
  Defines datasets, ownership, access, and preprocessing.

- `infra.yml`  
  Describes federated infrastructure: domains, registries, worker layout.

- `node.yml`  
  Binds logical components to physical nodes and interfaces.

Together, these artifacts allow Brane to discover code and data, plan and execute workflows across domains, and respect governance and infrastructure boundaries.