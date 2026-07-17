# Brane Documentation

Welcome to the Brane documentation. This repository contains three main document sets:

- **Brane Specification** – internal architecture and design for core developers.  
- **Brane User Guide** – practical usage guide organized by user role (system engineer, software developer, scientist, data policy expert, administrator).  
- **Brane Tutorials** – hands‑on exercises and workshop material (Hello World, pipelines, event scenarios).

> Note: CLI commands in these docs may still reflect older Brane versions; treat them as patterns and update them as needed for your deployment.

---

## 1. Brane Specification

Detailed technical description of Brane’s architecture, data and configuration model, workflow internal representation (WIR), virtual machine (VM), and language frontends.

- Directory: `brane-spec/`  
- Entry point: [`brane-spec/index.md`](brane-spec/index.md)

---

## 2. Brane User Guide

Role‑based guide for people using and operating Brane:

- System engineers: deployment and infrastructure  
- Software engineers: package development (`container.yml`, ECUs)  
- Scientists: workflows, datasets, BraneScript/Bakery  
- Data policy experts / stewards: policies, checker, reasoner  
- Administrators: instances, credentials, multi‑domain use

- Directory: `brane-user-guide/`  
- Entry point: [`brane-user-guide/index.md`](brane-user-guide/index.md)

---

## 3. Brane Tutorials

Hands‑on tutorials and scenario notes used in demos and workshops:

- Hello World package tutorial  
- Disaster Tweets workflow tutorial  
- Scenario: ICT.OPEN 2023  
- Scenario: UMC Utrecht demo & workshop

- Directory: `brane-tutorials/`  
- Entry point: [`brane-tutorials/index.md`](brane-tutorials/index.md)

---

## 4. extra documents

1. [Brane - core concepts - a not so short introduction to brane](brane-introduction.pdf)
2. [Overiew Presentation](brane-overview.pdf)
3. Brane intro
<iframe width="560" height="315" src="https://www.youtube.com/embed/rfSZAmLppRg?si=t3mp-hnYWJNnc3J5" title="YouTube vide    o player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-pictur    e; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## 5. Conventions

- All documents are written in Markdown (`.md`).  
- Relative links assume this `docs/` layout:
  - `docs/index.md`
  - `docs/brane-spec/index.md`
  - `docs/brane-user-guide/index.md`
  - `docs/brane-tutorials/index.md`
- CLI examples may require alignment with the current Brane CLI (`brane` / `branectl`) before use in production or training.

Use this `index.md` as the main entry point when browsing the documentation in the Git repo or a rendered docs site.
