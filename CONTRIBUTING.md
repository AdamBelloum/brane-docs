# Documentation QA Checklist

Use this checklist when reviewing any documentation change against
brane-deployment. The implementation source of truth is brane-deployment.

## Before merging a documentation change

### CLI commands
- [ ] Every shell command has been run against the documented brane-deployment revision
- [ ] brane certs add invocations pass all three files: ca.pem, client.pem, client-key.pem
- [ ] brane instance add invocations include --unchecked for TLS-only gRPC nodes
- [ ] brane workflow run invocations include a use-case argument and a .bs file path
- [ ] No command uses a flag or subcommand removed in the documented revision

### Ports and services
- [ ] Central brane-api is documented at port 50051
- [ ] Central brane-drv is documented at port 50053
- [ ] Worker brane-reg is documented at port 50051
- [ ] Worker brane-chk is documented at port 50052
- [ ] No removed or renamed service appears as current

### Architecture
- [ ] brane-job is documented as sharing brane-chk network namespace
- [ ] brane-job reaches the checker through localhost, not a hostname
- [ ] Policy deliberation and policy store paths are documented as file mounts, not network endpoints

### Role capabilities
- [ ] Administrator guide covers: deploy, health check, certificate generation, token generation
- [ ] Policy Manager guide covers: upload, validate, activate, inspect history
- [ ] User guide covers: package build, package list, instance add, certs add, workflow run
- [ ] No role guide documents capabilities belonging to another role as primary actions

### Frontend navigation
- [ ] Page labels match homepage.py PAGES dict labels exactly
- [ ] No removed page appears as a primary sidebar destination
- [ ] Workstation setup is documented as internally accessible, not a primary sidebar item
- [ ] CLI cross-links are present for every documented UI action

### Sensitive material
- [ ] No documentation example contains a real private key, JWT, or certificate bundle
- [ ] Policy token generation examples use placeholder values

### Known platform limitation
- [ ] Remote workflow execution is documented as requiring a server node (not the Mac control shell)
  when the Mac CLI config path divergence is unresolved

## Release checklist

When a new brane-deployment revision is deployed:

1. Record the new revision SHA in docs/audits/YYYY-MM-DD-baseline.md
2. Run the infrastructure health check and attach the output
3. Re-run the end-to-end validation (hello_world workflow)
4. Update the revision reference in docs/brane-spec/02-architecture.md
5. Update the revision reference in docs/brane-user-guide/06-administrators.md
6. Check this QA checklist against any changed CLI flags or service ports
# Contributor Guidance — Maintaining Documentation Alignment

## Core principle

brane-deployment defines what is supported.
brane-docs must describe that implementation at a specific, recorded revision.

Never document a command, port, service, or UI page as current unless it
exists in the recorded brane-deployment revision.

## Repository roles

| Repository | Role |
|---|---|
| brane-deployment | Implementation source of truth |
| brane-docs | User-facing description of that implementation |

## When to update documentation

Update brane-docs whenever brane-deployment changes any of the following:

| Change type | Documentation files to update |
|---|---|
| CLI flag added, removed, or renamed | brane-user-guide/08-brane-tools.md, relevant role guide, QA checklist |
| Service port changed | brane-spec/02-architecture.md, brane-spec/03-components.md |
| New or removed Docker service | brane-spec/03-components.md, brane-user-guide/06-administrators.md |
| Certificate or token procedure changed | brane-user-guide/06-administrators.md, brane-user-guide/02-quick-start-endusers.md |
| Policy lifecycle changed | brane-user-guide/05-data-policy-experts.md |
| Frontend route added, removed, or renamed | brane-user-guide/09-control-center.md |
| Ansible inventory format changed | brane-user-guide/06-administrators.md |

## How to record a documentation revision

Every documentation update that reflects a brane-deployment change must
include a baseline audit note at docs/audits/YYYY-MM-DD-baseline.md.

The note must record:
- brane-deployment branch and commit SHA
- brane-docs branch and commit SHA at the time of the update
- Health check output or a reference to it
- Any known limitations or deferred items

## Commit message conventions

Use the following prefixes:

    docs(audit):      baseline records and validation reports
    docs(arch):       architecture and service topology
    docs(cli):        CLI command reference
    docs(admin):      administrator operations guide
    docs(policy):     policy manager guide
    docs(user):       user workflow guide
    docs(frontend):   Control Center interface guide
    docs(tutorial):   tutorials and examples
    docs(qa):         QA checklist and contributor guidance

## Handling deprecated material

- Remove deprecated commands rather than marking them as alternatives.
- If historical context is needed, add a clearly labelled Deprecated section
  at the bottom of the file with the revision in which the item was removed.
- Never present deprecated syntax in a code block without a deprecation notice.

## Known open items (as of 2026-08-20)

| Item | Status | Resolution path |
|---|---|---|
| Remote workflow execution from Mac control shell | Blocked — Mac CLI config path divergence | Validate from server node; document workaround in quick-start |
| brane certs add SAN warnings | Cosmetic — --domain override works | Document as expected in quick-start |
| brane instance add health check over TLS | --unchecked required | Already documented in quick-start |
| Phase 4C user workflow remote validation | Deferred | Complete from server node SSH session |
