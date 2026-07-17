# Brane Specification

> Audience: Brane core developers and contributors  
> Scope: This document describes the internal architecture, data and configuration models, runtime, and language interfaces of Brane. It is not a user guide.

---

## 2. Architectural Overview

### 2.1. Federated deployment model

A Brane deployment consists of:

- A **central orchestrator** that coordinates workflow execution.
- One or more **domains** that provide compute and data.
- A **network of registries** and configuration files that describe where components live and how they communicate.

Domains retain autonomy:

- They decide which packages and datasets they expose.
- They enforce their own data policies for access and movement.
- They maintain local audit logs of workflow activity.

The orchestrator:

- Receives workflows (in WIR form) from frontends.
- Plans which domains will execute which tasks.
- Tracks data dependencies and data movement.
- Records global audit information.

Brane assumes an underlying infrastructure with:

- Stable network routes between orchestrator and domains.
- Configured nodes for central services, workers, and proxies.
- Per-domain configuration that binds logical components to concrete hosts.

### 2.2. Roles and responsibilities

Brane distinguishes several roles to separate concerns:

#### Orchestrator

The orchestrator hosts central components responsible for:

- Accepting workflows from frontends or APIs.
- Planning execution across domains.
- Coordinating data access and movement.
- Maintaining a global view of workflow progress and audits.

#### Domains (sites)

Each domain is responsible for:

- Executing tasks locally (through workers).
- Managing local datasets and intermediate results.
- Enforcing domain-specific data-use policies.
- Maintaining a **local audit log** of activity on that domain.

Domains integrate with Brane through well-defined interfaces:

- They expose workers and proxies to the orchestrator.
- They register packages and datasets.
- They implement policy checks for data movement and access.

#### Registry and policy services

Registries and policy-aware services:

- Track available packages and datasets.
- Resolve references in workflows to actual artifacts.
- Mediate data transfers between domains, respecting:
  - Ownership metadata.
  - Domain-specific rules.
  - Global policies when applicable.

### 2.3. Logical component model

Brane’s architecture is modular. Components are grouped into:

- **Orchestrator-side components**
- **Domain-side components**

#### 2.3.1. Orchestrator-side components

- **Driver**  
  The driver is the entry point for workflow execution. It receives workflows, delegates planning to the planner, and coordinates with domains to run tasks.

- **Planner**  
  The planner analyzes workflows, resolves data dependencies, and assigns tasks to domains. It decides when data must move between domains and how to respect policies.

- **Global audit log**  
  The global audit log records cross-domain activity:
  - Workflow submissions.
  - Task assignments.
  - Data movements between domains.

  It complements local audit logs, which are maintained by domains.

- **Orchestrator proxy**  
  The proxy abstracts communication between the orchestrator and domain-side components. It handles network details and routing so that higher-level components can focus on logic.

#### 2.3.2. Domain-side components

- **Worker**  
  A worker executes tasks on a domain’s compute resources. It:
  - Receives task descriptions from the orchestrator.
  - Runs code packaged in ECUs.
  - Produces intermediate results and datasets.

- **Local audit log**  
  The local audit log records domain-level activity. It is controlled by the domain and can follow domain-specific retention and access rules.

- **Checker**  
  The checker enforces domain policies for data access and movement. It decides:
  - Whether a workflow may read or write a given dataset.
  - Whether data can be transferred to another domain.
  - Under which conditions a data transfer is allowed.

- **Domain proxy**  
  Similar to the orchestrator proxy, the domain proxy manages domain-side communication and exposures of services (workers, registries, etc.) to the outside world.

### 2.4. Data and workflow model (high level)

Brane treats workflows as **data-dependency graphs**:

- Nodes represent tasks (package functions) or control-flow constructs.
- Edges represent dependencies between tasks and data.

From the orchestrator’s perspective:

- Each workflow has a set of required datasets and a set of produced results.
- Tasks may execute on different domains depending on:
  - Where required data resides.
  - Which domain hosts the needed package.
  - Policy constraints on data movement.

Data movement is:

- Explicit in the internal representation (WIR).
- Governed by domain policies and configured mechanisms:
  - Local access (data already present).
  - Transfers requested via preprocess kinds (e.g., registry-based transfers).

The rest of this document explains how these concepts are represented concretely in configuration files, internal data structures, and runtime behaviour.

