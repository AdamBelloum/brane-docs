# Quick Start for Brane Users

Use this page as an entry point to the current role-specific workflow.

## Before you begin

Ask an Administrator for:

- the central-node address;
- the Brane instance name;
- the certificate bundle for the domain you need to use.

Confirm that `brane` and Docker are available on your workstation:

```sh
brane --version
docker --version
```

## The user workflow

1. Discover local package and dataset sources.
2. Configure and select a Brane instance.
3. Register the Administrator-supplied certificate bundle.
4. Build and test a package when required.
5. Run a workflow locally.
6. Submit remotely only after selecting the intended instance.
7. Monitor the task and use Task History for persistent logs.

Start with the [Brane User Workflow Guide](03-endusers-scientists.md).

## First local workflow

The included `hello_world` package provides the smallest current local example:

```sh
cd /path/to/brane-deployment

brane package build --arch x86_64 packages/hello_world/container.yml
brane package test hello_world
brane workflow run <WORKFLOW_USER> packages/hello_world/hello_world.bs
```

On Apple Silicon, use `scripts/package_build_macOS.sh` instead of invoking the build command directly.

See the complete [Hello, World! tutorial](../brane-tutorials/02-hello-world-example.md).

## Remote-workflow prerequisite

A configured instance and registered certificate bundle establish connectivity and identity. They do not, by themselves, grant access to data or execution. The target domain policy must also permit the workflow.

For policy-denied submissions, contact the Policy Manager. For connection or infrastructure failures, contact the Administrator.

---
*Last verified against [brane-deployment](https://github.com/AdamBelloum/brane-deployment) @ `369392b9` on 2026-08-20.*
