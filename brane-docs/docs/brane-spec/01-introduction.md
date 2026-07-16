# Brane Specification

> Audience: Brane core developers and contributors  
> Scope: This document describes the internal architecture, data and configuration models, runtime, and language interfaces of Brane. It is not a user guide.

---

## 1. Introduction

### 1.1. What is Brane?

Brane is a federated middleware framework for executing data-centric workflows across multiple compute sites.

A typical Brane deployment connects:

- One **orchestrator**: the central coordination service.
- Multiple **domains** (sites): independent environments that own and control their data and compute resources.

Brane sits between:

- A scientist’s workflow (expressed in a frontend language such as BraneScript), and  
- A heterogeneous set of compute facilities (HPC clusters, local servers, cloud resources).

The framework focuses on:

- Executing workflows that span domains.
- Respecting data locality and data governance.
- Providing clear contracts for how domains participate in federation.

Brane is a **framework**, not a single fixed application. Each deployment can tailor Brane to a specific use case (for example, privacy-sensitive healthcare workflows) while still adhering to the core interfaces and behaviour defined here.

### 1.2. Design goals and non-goals

Brane’s design is driven by environments where:

- Data is distributed across sites.
- Each site controls access and movement of its data.
- Workflows need to cross site boundaries without centralising all data.

Key goals:

- **Federation before centralization**  
  Workflows are executed across domains. Brane avoids hard central control over data and computation where possible. Domains remain autonomous and enforce their own policies.

- **Data governance as a first-class concern**  
  Data ownership, policies, and auditability are central to Brane’s architecture. Domains decide which data can move, to where, and under what conditions.

- **Unified runtime across heterogeneous frontends**  
  Brane provides a common internal representation of workflows (WIR) and a shared runtime (VM). Frontend languages compile to WIR instead of embedding ad-hoc execution logic.

- **Separation of concerns for contributors**  
  Different parts of the system have clear responsibilities:
  - Orchestrator: planning, coordination, and global audits.
  - Domains: local execution, local audits, and policy enforcement.
  - Language frontends: user-facing syntax compiled into WIR.

Non-goals:

- Brane is **not** a general-purpose HPC scheduler for single-site workloads.
- Brane is **not** a GUI tool or end-user application; it is the middleware behind such tools.
- Brane does **not** replace site-level security or data policy systems; it integrates with them and respects their decisions.

### 1.3. Core concepts and terminology

To avoid ambiguity, we use the following terms consistently:

- **Domain**  
  A domain is an independently operated site that participates in a Brane deployment. Each domain owns its data, compute resources, and policies.

- **Orchestrator**  
  The orchestrator is the central component that coordinates workflows across domains. It does not own data; it plans and tracks execution.

- **Workflow**  
  A workflow is a data-centric computation described as a dependency graph. It may run tasks on multiple domains and move data between them when allowed.

- **Package**  
  A package bundles executable code and metadata into an **Executable Code Unit (ECU)**. Packages are described via `container.yml`.

- **Dataset**  
  A dataset represents data that workflows can use. Datasets are described via `data.yml`, which defines how the data is accessed and, when necessary, transferred.

- **Workflow Internal Representation (WIR)**  
  WIR is Brane’s internal graph-based representation of workflows. All frontends compile to WIR; the runtime executes WIR graphs.

- **Brane Virtual Machine (VM)**  
  The VM is the execution engine that interprets WIR graphs. It manages stacks, variables, frames, and external calls.

- **Registry**  
  The registry tracks packages and datasets for domains. Registries allow the orchestrator and domains to discover and reference shared artifacts.

### 1.4. Document structure

This document is organized for developers who work on Brane’s core and its extensions:

- **Section 2 – Architectural Overview**  
  High-level deployment model and core components.

- **Section 3 – Component Specifications**  
  Detailed contracts and behaviour for each major component.

- **Section 4 – Data and Configuration Model**  
  The structure and semantics of `container.yml`, `data.yml`, and infrastructure configuration files.

- **Section 5 – Workflow Internal Representation (WIR)**  
  How workflows are represented and manipulated internally.

- **Section 6 – Virtual Machine (VM) Design**  
  Runtime model, core data structures, and execution semantics.

- **Section 7 – BraneScript Frontend**  
  The BraneScript language as a frontend to WIR.

- **Section 8 – Extensibility and Future Work**  
  Extension points and planned evolution.

---
