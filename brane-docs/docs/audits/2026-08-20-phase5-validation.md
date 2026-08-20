# Phase 5 End-to-End Validation — 2026-08-20

## Baseline

| Repository | Revision |
|---|---|
| `brane-deployment` | `5cc39c4` (chore/ci-lint-baseline) |
| `brane-docs` | `386ed8fb` (main) |

## Local workflow path — PASSED

Executed on: Mac control shell (sp-byodm-145-3-67-57)

    brane package list
    # hello_world   1.0.0  ecu  OK
    # python_hello  1.0.0  ecu  OK

    brane workflow run tutorial-validator packages/hello_world/hello_world.bs
    # Hello, world!  OK

## Remote workflow path — BLOCKED (Mac CLI limitation)

### What works

    brane instance add ab-02.lab.uvalight.net \
      --name adam --user xin --use --unchecked --force
    # Successfully added new instance adam  OK

    brane certs add \
      --instance adam --domain ab-02.lab.uvalight.net --force \
      certs/central/ca.pem certs/central/client.pem certs/central/client-key.pem
    # Successfully added certificates for domain ab-02.lab.uvalight.net  OK

### What fails

brane package push and brane workflow run --remote both fail with:

    invalid HTTP version parsed

### Root cause

The Mac CLI reads the active instance from ~/.config/brane/instances.yml.
That file contains a single stale entry (remote at http://145.100.135.209).
brane instance select writes to ~/Library/Application Support/brane/ on macOS.
These two paths diverge: the CLI always routes remote calls through the stale
HTTP/1.1 entry, not the newly registered instance.

The deployed brane-api on port 50051 requires TLS gRPC (HTTP/2).
The Mac CLI sends HTTP/1.1, which the server rejects.

### Workaround

Remote push and workflow execution must be performed from a server node where
brane is installed at ~/.local/bin/brane and the config path is consistent.

## Tutorial corrections identified

| # | File | Correction |
|---|---|---|
| 1 | 02-hello-world-example.md | brane certs add must pass all three files: ca.pem, client.pem, client-key.pem. Single-file invocation fails. |
| 2 | 02-hello-world-example.md | brane instance add requires --unchecked when the node uses TLS-only gRPC. |
| 3 | 02-hello-world-example.md | Remote execution must be performed from a server node, not the Mac control shell. |
| 4 | 02-quick-start-endusers.md | Same corrections as above for the quick-start remote path. |
| 5 | Both files | SAN warnings on brane certs add are expected and cosmetic; --domain is the required override. |

## Status

| Path | Status |
|---|---|
| Local build + test + run | PASSED |
| Instance registration | PASSED |
| Certificate registration | PASSED |
| Remote push + run | BLOCKED — Mac CLI config path divergence |
