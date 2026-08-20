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
