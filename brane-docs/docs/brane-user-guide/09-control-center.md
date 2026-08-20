# Brane Control Center — Interface Guide

This guide describes the Brane Control Center web interface.
It reflects the implementation at brane-deployment revision 5cc39c4.

The Control Center is a Streamlit application served from the central node.
It provides the same operational capabilities as the shell helper scripts,
presented as a role-oriented graphical interface.

---

## Sidebar navigation

The sidebar organises the interface into three role workspaces and two
global destinations.

    Brane Control Center

    Home

    Administration (expander)
      Admin overview          [page: admin_overview]

    Policy management (expander)
      Policy overview         [page: policy_overview]

    User workspace (expander)
      User overview           [page: user_overview]
      Workflow editor         [page: user_workflows]

    ─────────────────────────
    Task history              [page: task_history]
    ─────────────────────────

    Active operations sidebar (always visible)

    Resources
      Brane website
      Documentation
      Source repository

The Administration and Policy management expanders open automatically
when the active page belongs to that workspace.
The active-operations sidebar remains visible regardless of which page is open.

---

## Home

**Route:** home
**Role:** any

A landing page with orientation links to each workspace.

---

## Administration workspace

**Route:** admin_overview
**Role:** administrator

The Admin overview is the single entry point for infrastructure operations.
It exposes four primary actions:

| Action | Purpose | Equivalent shell command |
|---|---|---|
| Deploy infrastructure | Run the Ansible deployment playbook | admin_helper.sh > deploy |
| Run health check | Verify central and worker containers, ports, and mounts | admin_helper.sh > health check |
| Domain certificates | Generate and download a client certificate bundle for a domain | generate_certs.sh |
| Policy-manager tokens | Generate and download a signed JWT for a policy manager | admin_helper.sh > generate token |

Long-running actions (deploy, health check) start a background task and
display progress through the task monitor. The task result is also available
in Task history.

Sensitive material (certificate private keys, policy tokens) is delivered
as a download and is not rendered in task logs or page output.

Advanced diagnostics and raw output are collapsed by default.

### Actions not in the primary Admin view

Infrastructure and cluster configuration details are accessible from within
the deployment flow, not as separate primary sidebar destinations.

---

## Policy management workspace

**Route:** policy_overview
**Role:** policy manager

The Policy overview presents the policy lifecycle in operational order:
prepare, validate, activate, inspect history.

| Action | Purpose | Equivalent shell command |
|---|---|---|
| Upload or edit policy | Add or modify an eFLINT policy file | policy_helper.sh > upload |
| Validate policy | Check the policy before activation | policy_helper.sh > validate |
| Activate policy | Make the policy active for a domain | policy_helper.sh > activate |
| Policy history | Inspect previously activated policies | policy_helper.sh > list |

Policy activation is visually distinct from validation and requires
an explicit confirmation step.

Policy credentials and tokens are not rendered in task logs or
browser-visible diagnostic output.

---

## User workspace

### User overview

**Route:** user_overview
**Role:** user

The User overview is the starting point for workflow work.

| Action | Purpose | Equivalent shell command |
|---|---|---|
| Test package or instance | Verify a package or instance is reachable | brane package test / brane instance list |
| Create workflow | Open the workflow editor | (see Workflow editor below) |
| Run workflow | Submit a workflow to the active instance | brane workflow run |

Status cards show available packages, available datasets, and recent
workflow runs.

Workflow run states shown: submitted, running, completed, failed,
denied by policy.

A policy-denial result is presented differently from an infrastructure
or execution failure.

Certificate, token, deployment, and cluster-management controls are
absent from the User overview.

### Workflow editor

**Route:** user_workflows
**Role:** user

A text editor for writing and submitting BraneScript workflows.
Equivalent to preparing a .bs file and running:

    brane workflow run <use-case> <file.bs>

---

## Task history

**Route:** task_history
**Role:** any

A global log of all tasks started from any workspace in the current session.
Accessible from the sidebar at all times.

---

## Workstation setup

**Route:** workstation_setup
**Role:** any

A guided setup page for installing and configuring the brane CLI on a
local workstation. Not shown as a primary sidebar destination; accessible
internally from workspace flows.

Equivalent manual steps:

    # Install brane CLI
    # See: brane-docs/docs/brane-user-guide/02-quick-start-endusers.md

---

## Navigation notes

- Workspace switching is navigation only, not an access-control mechanism.
  An administrator who needs to test a workflow uses User workspace.
  An administrator who needs to manage policies uses Policy management.
- A navigation request made inside a dashboard (for example, a
  "Go to User Workspace" button) is preserved across Streamlit reruns
  through session state.
- The active-operations sidebar remains visible during workspace switches
  so that long-running tasks are never hidden.

---

## Implementation reference

| Module | Responsibility |
|---|---|
| frontend/homepage.py | Sidebar, routing, session state initialisation |
| frontend/modules/admin_dashboard.py | Admin overview page |
| frontend/modules/policy_manager_dashboard.py | Policy overview page |
| frontend/modules/user_dashboard.py | User overview page |
| frontend/modules/Editor_Brane_Scripts.py | Workflow editor page |
| frontend/modules/task_ui.py | Active-operations sidebar and Task history |
| frontend/modules/Deploy_Infrastructure.py | Infrastructure deployment flow |
| frontend/modules/Cluster_Configurator.py | Cluster configuration flow |
| frontend/modules/Deploy_cli.py | Workstation setup page |
