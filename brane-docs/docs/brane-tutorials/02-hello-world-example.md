# Hello, World!

This tutorial uses the current `hello_world` source included with `brane-deployment`. It demonstrates a **local** package build, package test, and workflow run.

It does not claim a permitted remote execution. A remote run additionally requires a configured instance, a registered certificate bundle, and an active policy that permits the requested operation. That end-to-end path is validated separately.

## What you will use

The included package contains:

```text
packages/hello_world/
├── container.yml
├── hello_world.sh
├── hello_world.py
└── hello_world.bs
```

The package manifest defines:

- package name: `hello_world`;
- version: `1.0.0`;
- package kind: `ecu`;
- action: `hello_world`;
- one string output named `output`.

The included workflow is:

```branescript
import hello_world;

println(hello_world());
```

## Prerequisites

You need:

- `brane` available on `PATH`;
- Docker running;
- access to the `brane-deployment` checkout;
- an x86_64 package build target.

On Apple Silicon, use the supplied macOS build wrapper because deployment nodes use x86_64 packages.

## 1. Inspect the package source

```sh
cd /path/to/brane-deployment

find packages/hello_world -maxdepth 1 -type f -print | sort
```

Inspect the manifest and workflow if needed:

```sh
sed -n '1,220p' packages/hello_world/container.yml
sed -n '1,120p' packages/hello_world/hello_world.bs
```

## 2. Build the package

On Linux or another x86_64 build host:

```sh
brane package build --arch x86_64 packages/hello_world/container.yml
```

On Apple Silicon:

```sh
bash scripts/package_build_macOS.sh packages/hello_world/container.yml
```

The macOS wrapper verifies `docker` and `brane`, configures its Buildx builder when needed, and then builds for x86_64.

## 3. Confirm the package is available locally

```sh
brane package list
```

The output should include `hello_world` after a successful build.

## 4. Test the package

```sh
brane package test hello_world
```

Choose the `hello_world` action if prompted. The package has no input arguments. A successful test returns the action's string output.

If the package test fails, inspect the container build output before changing the workflow.

## 5. Run the local workflow

Choose a workflow label. This is your submission label and does not require Administrator-created user credentials.

```sh
brane workflow run <WORKFLOW_USER> packages/hello_world/hello_world.bs
```

The workflow imports the `hello_world` package, calls its action, and prints the returned value.

## 6. Continue to remote execution

Do not add endpoint or certificate flags directly to the workflow command.

Instead:

1. configure and select an instance;
2. register the appropriate certificate bundle;
3. confirm that the Policy Manager has activated a policy that permits the run;
4. submit using:

```sh
brane workflow run --remote <WORKFLOW_USER> packages/hello_world/hello_world.bs
```

A denial after remote submission can be expected when deny-all is active or the active policy does not allow the workflow. It is not necessarily an infrastructure failure.

For the complete user path, see the [Brane User Workflow Guide](../brane-user-guide/03-endusers-scientists.md).

---
*Last verified against [brane-deployment](https://github.com/AdamBelloum/brane-deployment) @ `369392b9` on 2026-08-20.*
