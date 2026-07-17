


## 1. Introduction

### 1.1. What is Brane?

Brane is a programmable orchestration framework for running **data-centric workflows** across multiple compute sites (domains). It gives scientists and engineers a way to:

- Compose workflows from reusable **packages** (containerized code units),
- Run those workflows across **distributed infrastructure**, and
- Respect **data governance and network policies** defined by domain owners.

A typical Brane deployment consists of:

- A **client** (Brane CLI) that users run on their own machines.
- A **Brane instance** (control node + worker nodes) that coordinates and executes workflows.

The instance provides:

- A central **driver** that receives workflows and issues jobs,
- A **planner** that decides which worker domain should run each task,
- A central **registry** that tracks packages, datasets, and domains,
- **Proxies** that handle secure access and certificate-based authentication.

Worker nodes host:

- A **delegate** that connects Brane to the local compute backend,
- A local **registry** for datasets and intermediate results,
- A **policy checker** (PEP) with a reasoner (e.g., eFLINT),
- A **local proxy** for secure communication.

### 1.2. Roles and responsibilities

Brane is designed for several user roles, each with different responsibilities.

- **System engineers**  
  Install and maintain Brane instances:
  - Configure control and worker nodes,
  - Set up proxies and certificates,
  - Integrate Brane with local compute backends.

- **Software engineers (package developers)**  
  Create and maintain Brane **packages**:
  - Author `container.yml` files,
  - Build container images,
  - Test packages locally,
  - Publish them to Brane instances for others to use.

- **Policy experts**  
  Define and maintain **data and network policies**:
  - Express rules in a reasoner (e.g., eFLINT),
  - Configure the policy checker,
  - Use tokens and tools to manage and test policy behaviour.

- **Scientists (workflow authors)**  
  Design and run workflows:
  - Write BraneScript or Bakery programs,
  - Import and call packages,
  - Work with datasets and intermediate results,
  - Interpret results and iterate.

- **Administrators**  
  Manage Brane instances and cross-domain usage:
  - Control instance access and credentials,
  - Manage client certificates and identities,
  - Coordinate interactions between domains.

This guide is organized around these roles, so each user can focus on the sections relevant to their work.

### 1.3. Brane components (user-level view)

From a user perspective, Brane has three major parts: 

1. **Central control node (Brane instance)**  
   - Runs the driver, planner, central registry, and central proxy.
   - Receives workflows from scientists and orchestrates work across domains.

2. **Worker nodes (domains/locations)**  
   - Run delegates and local backends (e.g., Docker-based, HPC, or custom).
   - Host local registries for datasets and intermediate results.
   - Enforce policies through local policy checkers.

3. **Client environment (your machine)**  
   - Hosts the Brane CLI.
   - Stores local packages, configuration, and certificates.
   - Sends workflows and package operations to Brane instances.

The remainder of this guide explains how each role interacts with these components.

### 1.4. How to use this guide

- If you are a **scientist** new to Brane:
  - Start with **Section 2 (Quick Start)**, then read **Section 6 (Authoring Workflows)**.

- If you are a **system engineer**:
  - Focus on **Section 3 (Deploying Brane)** and **Section 5.5 / 7.2 (Certificates and identities)**.

- If you are a **software engineer**:
  - Go to **Section 4 (Developing Packages)** and see examples in **Section 6**.

- If you are a **policy expert**:
  - Read **Section 5 (Policies and Checker)**.

- If you are an **administrator**:
  - Focus on **Section 7 (Instances and Credentials)** and **Section 8 (Reference)**.

Some command examples may still use older CLI syntax. We will keep them conceptually in place and **update them later**, based on deployment notes and current CLI versions.

---
