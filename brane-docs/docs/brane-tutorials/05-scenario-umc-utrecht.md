# 5. Scenario: UMC Utrecht Demo & Workshop

This chapter describes how the shared tutorials were used at the **UMC Utrecht** to conclude a Brane Proof-of-Concept (PoC) on pseudonymised patient data.

## 5.1. Context and goals

- Audience:
  - Clinicians,
  - Data stewards,
  - Technical staff and researchers.
- Goal:
  - Demonstrate Brane as a **data sharing infrastructure** for analysis on pseudonymised patient data.
  - Show how workflows and packages can be used in a regulated healthcare environment.

The tutorial targeted Brane framework **version 3.0.0**.

## 5.2. Session structure

The demo/workshop was split in two halves:

- **First half: SIG meeting (presentation)**
  - Introduced Brane conceptually:
    - Domains and workflows,
    - Packages and datasets,
    - Governance and policy.

- **Second half: hands‑on workshop**
  - Followed the [Hello World Package Tutorial](02-hello-world-package.md), including remote execution.
  - Demonstrated an existing pipeline (similar to the [Disaster Tweets Workflow](03-disaster-tweets-workflow.md), but using healthcare-specific data in the PoC).
  - Showed the technical setup of the PoC:
    - Brane instance across multiple domains,
    - Policy checker and pseudonymised datasets.

## 5.3. Differences from the generic tutorials

Key differences from ICT.OPEN and the generic tutorials:

- **Framework version**:
  - Materials and screenshots based on Brane **3.0.0**.
  - Some CLI commands and warnings differed from 2.x.

- **Focus on remote execution**:
  - Participants connected to a shared Brane instance (central node + worker domains).
  - Exercises included:
    - Adding the instance via `brane instance add`,
    - Adding client certificates for domains,
    - Running workflows with the `--remote` flag,
    - Using location annotations (`on "worker1" { ... }`) to decide where code runs.

- **Healthcare PoC setup**:
  - Datasets representing pseudonymised patient data.
  - Domain policies and data governance emphasized.
  - Discussion of how Brane’s policy checker and audit logs integrate with healthcare requirements.

- **Brane IDE (Jupyter-based)**:
  - Demonstrated an experimental BraneScript notebook interface:
    - JupyterLab-based environment,
    - BraneScript cells sent to the Brane instance using a shared CLI library.
  - Workflow authoring in notebooks mirrored the CLI-based tutorials, but with richer interactive experience.

## 5.4. Reuse

If you organize a UMC-style workshop today:

- Use the shared tutorials:
  - [Hello World Package](02-hello-world-package.md),
  - [Disaster Tweets Workflow](03-disaster-tweets-workflow.md) or a healthcare-specific pipeline.
- Add a local **PoC scenario**:
  - Prepare datasets and packages that reflect your healthcare use case.
  - Configure a Brane instance with multiple domains and policy enforcement.
- Emphasize:
  - Remote execution,
  - Governance (data policies, audit),
  - Integration with notebook-based tooling (e.g., updated Brane IDE or other frontends).