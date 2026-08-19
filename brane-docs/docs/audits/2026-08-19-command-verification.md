# Command Verification Log — 2026-08-19

## Scope and baseline

- **Documentation repository:** `brane-docs`
- **Implementation source of truth:** `brane-deployment`
- **Deployment baseline:** `main@369392b991e0c3290739077d0ad071b5ce3f76bb`
- **Verification date:** 2026-08-19
- **Evidence collected:** deployed `brane --help`, `branectl --help`, detailed subcommand help, deployment scripts, and a baseline health report.

The deployed CLIs were verified on the central node and both worker nodes. They are installed at:

```text
/home/adam/.local/bin/brane
/home/adam/.local/bin/branectl
```

They were not present at `/root/.local/bin`. Remote administration through the root SSH account must use the explicit paths above or execute commands as the deployed `adam` user.

## Status definitions

- **Verified and current:** command syntax was confirmed through deployed CLI help or direct script help.
- **Current; prerequisite-dependent:** syntax is confirmed, but successful execution depends on deployment state, credentials, configuration, or other inputs.
- **Deprecated or removed:** syntax does not exist in the deployed CLI command tree.
- **Requires an operational test:** syntax is known, but a specific end-to-end behaviour has not yet been tested.

## Verified command reference

| Command family | Status | Prerequisites | Expected result |
|---|---|---|---|
| `brane package build [--arch ARCH] <FILE>` | Current; prerequisite-dependent | Package manifest; Docker; compatible architecture; use `--arch x86_64` when building on Apple Silicon for the x86_64 deployment nodes. | Builds a local Brane package. |
| `brane package import [--arch ARCH] <REPO> [FILE]` | Current; prerequisite-dependent | Reachable Git repository and valid package file path relative to the repository. | Imports and builds the selected package. |
| `brane package list [--latest]` | Current; prerequisite-dependent | Local Brane package state. | Lists locally known packages. |
| `brane package test <NAME> [VERSION]` | Current; prerequisite-dependent | Locally available package and Docker access. | Runs a package test locally. |
| `brane package push [NAME[:VERSION] ...]` | Current; prerequisite-dependent | Selected remote instance, registry access, and a local package. | Pushes one or more packages to the registry. |
| `brane data build <FILE>` | Current; prerequisite-dependent | Valid `data.yml` and associated dataset files. | Registers a locally available dataset. |
| `brane data list` | Current; prerequisite-dependent | Local Brane data state. | Lists locally known datasets. |
| `brane instance add <HOSTNAME> [--name NAME] [--use]` | Current; prerequisite-dependent | Reachable instance hostname and, where required, valid domain certificates. | Registers an instance and optionally selects it. |
| `brane instance list [--show-status]` | Verified and current | Local Brane instance configuration. | Lists registered instances; optional status performs reachability checks. |
| `brane instance select <NAME>` | Current; prerequisite-dependent | A registered instance with the given name. | Selects the named instance. |
| `brane certs add [PATHS ...] [--domain DOMAIN]` | Current; prerequisite-dependent | CA certificate, signed client certificate, and associated private-key material supplied for the target domain. | Adds certificate material to an instance configuration. |
| `brane certs list [--all] [--instance INSTANCE]` | Verified and current | Local Brane instance configuration. | Lists registered certificate domains. |
| `brane workflow run <USE_CASE> <FILE> [--remote]` | Requires an operational test | Use-case registry, valid workflow, required packages and datasets; selected instance and certificates for `--remote`. | Runs the workflow locally or submits it remotely. |
| `brane workflow run <USE_CASE> <FILE> --dry-run` | Current; prerequisite-dependent | Use-case registry and valid workflow. | Runs the BraneScript layer with dummy task results; real task containers are not executed. |
| `brane workflow check <FILE> [--user USER]` | Current; prerequisite-dependent | Selected remote instance and policy-checker connectivity. | Checks workflow policy without executing the workflow. |
| `brane workflow repl <USE_CASE> [--remote]` | Current; prerequisite-dependent | Use-case registry; selected instance for remote use. | Starts an interactive BraneScript session. |
| `branectl policies add <INPUT> [--language eflint]` | Current; prerequisite-dependent | Policy file, checker connectivity, and authorised policy-manager credentials. | Adds a policy version without activating it. |
| `branectl policies list` | Current; prerequisite-dependent | Checker connectivity and authorised policy-manager credentials. | Lists policies available on the checker. |
| `branectl policies activate [VERSION]` | Current; prerequisite-dependent | Existing policy version, checker connectivity, and authorised policy-manager credentials. | Activates the selected policy version. |
| `branectl generate policy_token <INITIATOR> <SYSTEM> <DURATION>` | Current; protected operation | Administrator-controlled policy secret and a safe output path. | Creates a time-limited policy-manager JWT. |
| `bash scripts/brane_gen_cert.sh --help` | Verified and current | Repository checkout and Bash. | Displays certificate-generation options. |
| `bash ../scripts/brane_healthcheck.sh --report` | Current; prerequisite-dependent | Run from `docker-deployment/`; deployment virtual environment on `PATH`; valid inventory; administrator SSH access. | Produces a read-only central and worker health report. |
| `bash scripts/brane_cleanup.sh --inventory PATH` | Current; prerequisite-dependent | Valid inventory, SSH access, and explicit confirmation of destructive impact. | Removes deployment Docker resources and deployment directories. |

## Deprecated command migration

The following top-level forms do not exist in the deployed `brane --help` command tree. They are retained here only as historical migration information.

| Deprecated form | Current replacement |
|---|---|
| `brane build <FILE>` | `brane package build <FILE>` |
| `brane list` | `brane package list` |
| `brane import <REPO> [FILE]` | `brane package import <REPO> [FILE]` |
| `brane test <NAME>` | `brane package test <NAME> [VERSION]` |
| `brane run <FILE>` | `brane workflow run <USE_CASE> <FILE>` |
| `brane repl` | `brane workflow repl <USE_CASE>` |

## Operational and security notes

- `brane workflow run` requires both a use-case argument and a workflow-file argument.
- Remote workflow execution uses `--remote` after the required arguments. Configure instances and certificates separately; `workflow run` does not support `--cert` or `--key` options.
- Do not place policy JWT values, private keys, certificate bundles, or secret-file contents in documentation or shell-history examples.
- The default deployment policy is deny-all. A remote workflow can be correctly submitted yet denied until an appropriate domain policy has been uploaded and activated.
- Package builds, policy uploads and activation, and remote workflow execution require controlled end-to-end validation before being described as tested workflows.

## Unsupported help invocations

Do not document these forms as valid help commands:

| Invocation | Observed behaviour |
|---|---|
| `scripts/brane_healthcheck.sh --help` | Treats `--help` as unknown and then attempts the default inventory path. |
| `scripts/brane_cleanup.sh --help` | Treats `--help` as unknown. |
| `scripts/show_brane_tools_help.sh --help` | Expects positional `brane` or `branectl`, not `--help`. |
