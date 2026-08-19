# Phase 1 — Implementation-to-Documentation Traceability Matrix

**Audit date:** 2026-08-19
**Documentation baseline:** `brane-docs` `main@7a15f8c2e50d74775668382aa916cea59fe0220a`
**Implementation baseline:** `brane-deployment` `main@369392b991e0c3290739077d0ad071b5ce3f76bb`
**Health baseline:** passed; infrastructure health-check exit code `0`
**Phase 1 status:** Complete; provenance classifications are recorded in `2026-08-19-documentation-provenance.md`.

## Status labels

- **Verified** — directly confirmed against the baseline implementation or Phase 0 health check.
- **Discovered** — present in the baseline source tree; interface or runtime behaviour still requires verification.
- **Requires verification** — current documentation cannot yet be retained as operationally accurate.
- **Historical/provenance review** — page is not sourced by the current deployment repository; determine and record its authoritative upstream source, or label it archival.
- **Undocumented** — a current implementation surface lacks a MkDocs documentation location.

## Baseline deployment model

The verified baseline deployment mode is Docker/Ansible using `docker-deployment/site.yml` and the local, untracked `docker-deployment/inventories/production/hosts.ini`.

- `site.yml` tags: `prerequisites`, `branectl`, `branecli`, `workers`, `central`, `certs`, `start`, and `smoke`.
- `deploy_docker.yml` references role names not present in the baseline inventory and is therefore **legacy or unverified** pending a syntax check.
- The implementation downloads Compose templates and smoke-test material from the repository `main` branch. The baseline commit alone does not freeze those remotely fetched artefacts.
- The Compose templates and generated environment files assume two named workers (`worker-a`, `worker-b`) and use `groups['workers'][0]` and `[1]`. Any claim of arbitrary worker-count support requires verification.

## Implementation-to-documentation mapping

| Implementation source | User-facing purpose | Current MkDocs location | Status | Required action |
|---|---|---|---|---|
| `scripts/brane_main.sh` | Role selection and dispatch | None | Undocumented | Add a concise operational-entry-points page or incorporate into role-guide introductions. |
| `scripts/brane_helper_admin.sh` | Deployment lifecycle, health, certificates, tokens, instances, packages, workflows, datasets | `brane-user-guide/06-administrators.md` | Requires verification | Replace unverified manual procedures with supported helper/Ansible workflow after Phase 3 command audit. |
| `scripts/brane_healthcheck.sh` | Read-only central/worker health report | `brane-user-guide/06-administrators.md` | Verified entry point; undocumented interface | Document required working directory, inventory, SSH/Ansible prerequisites, options, output, and exit codes. |
| `scripts/brane_gen_cert.sh` | Generate client certificates | `brane-user-guide/06-administrators.md` | Discovered | Verify arguments, certificate layout, signing and secure distribution process. |
| `scripts/brane_cleanup.sh`, `scripts/clean_central_worker.sh` | Destructive cleanup | None | Undocumented | Document only after scope, confirmation, and preservation behaviour are verified. |
| `scripts/package_build_macOS.sh` | macOS package build | `brane-user-guide/04-software-engineers.md` | Discovered | Document x86_64 target requirement and output path. |
| `scripts/troubleshoot-brane-deployment.sh` | Diagnostics | None | Undocumented | Verify safe diagnostic use and map to troubleshooting reference. |
| `scripts/smoke-test/run-smoke-test.sh` | Deployment smoke test | Tutorials / Administrator guide | Discovered | Verify prerequisites, expected outcome, and safe invocation. |
| `scripts/brane_helper_policy.sh` | Token/environment validation, policy addition, policy activation | `brane-user-guide/05-data-policy-experts.md` | Requires verification | Rewrite as prepare/inspect/upload/activate workflow; document deny-all baseline. |
| `scripts/brane_helper_user.sh` | Environment, instance, certificate registration, package actions, local/remote workflow run | `brane-user-guide/03-endusers-scientists.md` | Requires verification | Map each helper action to current CLI syntax and prerequisites. |
| `docker-deployment/site.yml` and `roles/brane_deploy/` | Current Docker/Ansible deployment | `brane-user-guide/06-administrators.md`, `brane-spec/02-architecture.md` | Discovered | Create deployment and topology reference; document supported tags and inventory setup. |
| `scripts/templates/central-compose.yml` | Central Compose topology | `brane-spec/02-architecture.md` | Discovered | Document central services, ports, mounts, dependencies and generated paths. |
| `scripts/templates/worker-compose.yml` | Worker Compose topology | `brane-spec/02-architecture.md` | Discovered | Document registry/checker/job topology, policy configuration and Docker socket mount. |
| `docker-deployment/group_vars/all.yml` and role defaults | Version, path, endpoint and download configuration | `brane-spec/04-data-config.md`, Administrator guide | Requires verification | Correct endpoint table, identify unused/conflicting variables and explain overrides. |
| `frontend/homepage.py` | Streamlit routing and navigation | None | Undocumented | Add frontend guide with actual routes and role boundaries. |
| `frontend/modules/admin_dashboard.py` | Deploy, health, certificates, policy tokens | None | Undocumented | Document four primary actions, task behaviour and secure downloads. |
| `frontend/modules/policy_manager_dashboard.py` | Policy inspection/upload/activation/token state | None | Undocumented | Document policy lifecycle and deny-all outcome. |
| `frontend/modules/user_dashboard.py` | Instances, packages, certificate registration, workflow preparation | `brane-user-guide/03-endusers-scientists.md` | Requires verification | Align guide to dashboard workflow and role boundaries. |
| `frontend/modules/Editor_Brane_Scripts.py` | Workflow Studio | `brane-user-guide/03-endusers-scientists.md`, `07-branscript.md` | Requires verification | Document local vs remote submission and selected-instance behaviour. |
| `frontend/modules/task_ui.py` | Global activity sidebar and persisted task history | None | Undocumented | Document task states/history; audit displayed logs and metadata for secret redaction. |
| `certs/` | Local certificate bundles | Administrator and User guides | Discovered | Document layout, generation, registration and secure handling; never publish private material. |
| `policies/`, `policy_tokens/` | Local eFLINT policies and policy-manager tokens | Policy guide | Discovered | Document file handling, expiry, upload, inspection and activation. |
| `packages/`, `datasets/` | User package and dataset resources | User and Developer guides | Discovered | Document discovery, package architecture/build, and dataset layout. |
| `frontend/runtime/tasks/` | Generated local task records/logs | None | Undocumented | Define retention, diagnostic use, and secret-handling rules. |
| `k8-deployment/` | Kubernetes/Helm deployment material | None | Unverified deployment mode | Classify as supported, deprecated, or archival before documenting it as an option. |

## Verified topology facts for Phase 2

| Location | Service | Connectivity / configuration |
|---|---|---|
| Central | `brane-api` | Compose publishes `${API_PORT}`; active default is `50051`. |
| Central | `brane-drv` | Compose publishes `${DRV_PORT}`; active default is `50053`. |
| Central | `brane-prx`, `brane-plr` | Present; no host port published in the local template. |
| Central | `aux-scylla`, `aux-kafka`, `aux-zookeeper` | Internal auxiliary services. |
| Worker | `brane-reg` | Publishes port `50051`. |
| Worker | `brane-chk-<location_id>` | Publishes port `50052`; receives policy deliberation and policy-store secret-file paths. |
| Worker | `brane-job-<location_id>` | Shares the checker network namespace via `network_mode: "service:brane-chk"`; node configuration reaches checker at `localhost`. |

### Endpoint conflict requiring correction

`group_vars/all.yml` declares `brane_hub_api` using central port `30051`, while the active Compose/default configuration publishes `brane-api` on port `50051`. Do not document port `30051` as the current API endpoint until a controlled runtime test explains or resolves the conflict. `brane_registry_url` also targets central port `50051`, but its intended service semantics require verification.

## Frontend traceability findings

### Current routes

- Home
- Workstation setup
- Administration: Admin overview, Infrastructure, Cluster configuration
- Policy management: Policy overview
- User workspace: User overview, Workflow editor
- Task history

### Documentation and implementation discrepancies

1. `frontend/README.md` describes an older administrator-centred navigation model and references pages that are not current primary routes.
2. The sidebar still exposes `Infrastructure` and `Cluster configuration` as direct Administration buttons. The intended role-oriented design calls for those capabilities to be embedded in deployment flow or advanced settings; current documentation must describe implementation, not planned design.
3. The Admin dashboard has four primary actions: deployment, health check, domain certificates, and policy-manager tokens.
4. Policy Management exposes inspect, upload, list, activate, token status and SSH connectivity. It warns that deny-all remains in effect until an appropriate policy is activated.
5. User Workspace currently exposes certificate registration. This conflicts with the intended information architecture that reserves certificate controls for administration; resolve implementation/design alignment before describing it as role policy.
6. Task UI displays task logs for active, failed and interrupted tasks. Secret-redaction behaviour requires a dedicated audit.

## Documentation-page provenance and action map

| Documentation page(s) | Identified implementation source | Classification / required action |
|---|---|---|
| `docs/index.md`, `docs/faq.md` | No current implementation source identified | Review against current Docker/Ansible, CLI and frontend surfaces; update or label historical. |
| `docs/brane-user-guide/index.md` | Role helpers and Streamlit role workspaces | Rewrite navigation and role overview after Phases 3, 4 and 6. |
| `docs/brane-user-guide/01-introduction.md`, `02-quick-start-endusers.md` | No verified current source; pages lack H1 titles | Establish current quick-start reference path in Phase 5; add proper titles. |
| `docs/brane-user-guide/03-endusers-scientists.md` | `brane_helper_user.sh`, `user_dashboard.py`, `Editor_Brane_Scripts.py` | Correct commands and task flow in Phases 3–5. |
| `docs/brane-user-guide/04-software-engineers.md` | `package_build_macOS.sh`, `packages/`, Brane CLI | Verify package commands and Apple-Silicon/x86_64 requirements. |
| `docs/brane-user-guide/05-data-policy-experts.md` | `brane_helper_policy.sh`, `policy_manager_dashboard.py`, policy task modules | Correct token, upload, inspection and activation workflow. |
| `docs/brane-user-guide/06-administrators.md` | `brane_helper_admin.sh`, `brane_healthcheck.sh`, `site.yml`, Ansible role/tasks | Replace legacy/unverified direct deployment procedures with supported operations. |
| `docs/brane-user-guide/07-branscript.md` | `Editor_Brane_Scripts.py`; upstream BraneScript language implementation | Verify language-reference provenance and current syntax. |
| `docs/brane-user-guide/08-brane-tools.md` | Ansible CLI installation tasks, Workstation setup | Add to MkDocs navigation or consolidate; verify local/remote installation scope. |
| `docs/brane-spec/index.md`, `01-introduction.md`–`09-references.md` | Upstream Brane Framework specification source, revision unknown | Pin authoritative upstream revision or explicitly label as historical/reference material. Correct duplicate/misnumbered H1 titles in `01`, `02`, and `08`. |
| `docs/brane-tutorials/index.md`, `01-overview.md` | Tutorial collection, current source unknown | Classify current versus archival after Phase 5 reference-path validation. |
| `docs/brane-tutorials/02-hello-world-example.md`, `03-disaster-tweets-example.md`, `04-scenario-ictopen.md`, `05-scenario-umc-utrecht.md` | Example package/workflow sources not yet verified against baseline | Verify against current packages/datasets/workflows or label historical. |
| `docs/brane-tutorials/02-introduction.md`, `03-hello-world-part2.md` | No navigation entry and source unknown | Add to navigation only if retained and verified; otherwise consolidate or archive. |
| `docs/audits/2026-08-19-baseline.md` | Phase 0 baseline evidence | Current audit record; keep tracked and out of primary user navigation. |

## Navigation defects

The following pages are present but omitted from `mkdocs.yml`:

- `docs/brane-tutorials/02-introduction.md`
- `docs/brane-tutorials/03-hello-world-part2.md`
- `docs/brane-user-guide/08-brane-tools.md`

## Phase 1 closure condition

All user-facing implementation components now have a current documentation location or a clear documentation task. To close Phase 1 formally, record the authoritative source/revision or explicit archival status for:

1. the Brane specification pages;
2. the legacy/tutorial scenario pages;
3. the FAQ and introductory pages with no verified implementation source.

After that provenance decision, proceed in parallel with Phase 2 (architecture correction), Phase 3 (command audit), and Phase 6 (frontend documentation alignment).
