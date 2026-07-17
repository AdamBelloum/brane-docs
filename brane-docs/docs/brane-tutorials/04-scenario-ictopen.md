
### `04-scenarios-ictopen.md`

```markdown
# 4. Scenario: ICT.OPEN 2023

This chapter describes how the shared tutorials were used at **ICT.OPEN 2023**, the conference where Brane was first introduced via a hands‑on tutorial.

## 4.1. Context and goals

- Audience: mixed academic and industry participants.
- Goal:
  - Experience Brane both as a **package developer** and as a **scientist**,
  - Understand what “working with Brane” looks like in practice.

The tutorial targeted Brane framework **version 2.0.0**.

## 4.2. Tutorial structure

Scheduled blocks:

- **12:30–12:45** – Introduction (presentation)
  - High-level overview of Brane’s architecture and goals.

- **12:45–13:30** – Part 1: *Hello, World!* (guided hands‑on)
  - Followed the [Hello World Package Tutorial](02-hello-world-package.md).

- **13:30–13:45** – Break.

- **13:45–14:15** – Part 2: *A workflow for Disaster Tweets* (hands‑on)
  - Followed the [Disaster Tweets Workflow Tutorial](03-disaster-tweets-workflow.md).

- **14:15–14:30** – Evaluation and Q&A.

## 4.3. Differences from the generic tutorials

The shared tutorials were adapted slightly for ICT.OPEN:

- **Framework version**:
  - Tutorial material and screenshots were based on Brane **2.0.0**.
  - Some CLI options and warning messages reflect that version.

- **Execution mode**:
  - Most participants ran exercises **locally** on laptops:
    - Building and testing packages,
    - Running workflows in local mode.
  - Remote execution was demonstrated conceptually but not central.

- **Data and packages**:
  - Disaster Tweets datasets and packages (`compute`, `visualization`) were prepared ahead of time.
  - Participants used `brane import` and `brane data build` with provided URLs and archives.

## 4.4. Reuse

If you re-run an ICT.OPEN-style tutorial today:

- Use the shared tutorials:
  - [Hello World Package](02-hello-world-package.md),
  - [Disaster Tweets Workflow](03-disaster-tweets-workflow.md).
- Adjust time blocks as needed.
- Update CLI commands to match your current Brane version and environment.