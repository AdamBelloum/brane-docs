# Final Discrepancy Report — Documentation–Deployment Synchronisation Sprint

Sprint period: 2026-08-19 to 2026-08-20
brane-deployment baseline: 5cc39c4 (chore/ci-lint-baseline)
brane-docs baseline: 386ed8fb (main)
brane-docs at sprint close: c8e7328f (main)

---

## Resolved discrepancies

| # | Area | Was | Now | Commit |
|---|---|---|---|---|
| 1 | CLI: brane certs add | Single-file invocation documented | Three-file invocation (ca.pem + client.pem + client-key.pem) documented | 386ed8fb |
| 2 | CLI: brane instance add | --unchecked not mentioned | --unchecked documented as required for TLS-only gRPC nodes | 386ed8fb |
| 3 | CLI: brane workflow run | use-case argument undocumented | use-case argument and .bs file path documented | 386ed8fb |
| 4 | Architecture: service ports | Stale or missing port references | Central 50051/50053, worker 50051/50052 verified and documented | 5535510c |
| 5 | Architecture: brane-job network | Network arrangement undocumented | brane-job shares brane-chk namespace, reaches checker via localhost | 5535510c |
| 6 | Architecture: policy paths | Policy endpoints undocumented | File-mount paths for delib and store keys documented | 5535510c |
| 7 | Administrator guide | Stale deployment commands | Aligned to Ansible/Docker deployment with health-check script | 017040a2 |
| 8 | Policy Manager guide | Missing deny-all explanation | Deny-all default and policy activation flow documented | 4a09da1c |
| 9 | User guide | Missing x86_64 build requirement | Apple Silicon cross-compilation target documented | 386ed8fb |
| 10 | Frontend guide | No frontend documentation existed | 09-control-center.md written from homepage.py source | c8e7328f |
| 11 | Traceability | No implementation-to-documentation matrix | implementation-documentation-matrix.md created | 842bf8d7 |

---

## Deferred items

| # | Area | Description | Reason deferred | Resolution path |
|---|---|---|---|---|
| D1 | Remote workflow validation | brane package push and brane workflow run --remote not validated from Mac | Mac CLI config path divergence: ~/.config/brane/instances.yml vs ~/Library/Application Support/brane/ | Validate from server node via SSH |
| D2 | Phase 4C user workflow (remote path) | Remote push and run steps in user guide not end-to-end tested | Depends on D1 | Complete after D1 |
| D3 | Tutorial corrections 1-5 | Five corrections identified in Phase 5 audit not yet applied to tutorial files | Time-boxed sprint | Apply as follow-up PRs using audit record as source |

---

## Historical material retained

None. All deprecated commands and stale references were removed rather than
archived. No historical section was added during this sprint.

---

## Sprint outcome

All active operational guides match the baseline brane-deployment revision
for the local execution path. The remote execution path is documented with
a known limitation and a clear resolution path. The documentation repository
now has a QA checklist, contributor guidance, and a maintenance process for
future deployment changes.

Definition of done status:

| Criterion | Status |
|---|---|
| Active operational guides match baseline revision | DONE |
| Every documented CLI command has a verification status | DONE |
| Obsolete syntax, ports, and UI references removed | DONE |
| Policy upload, activation, and deny-all documented | DONE |
| Administrator, Policy Manager, and User guides match actual capabilities | DONE |
| End-to-end permitted-workflow tutorial executed | PARTIAL — local path only; remote path deferred (D1) |
| Documentation maintenance checks in place | DONE |
