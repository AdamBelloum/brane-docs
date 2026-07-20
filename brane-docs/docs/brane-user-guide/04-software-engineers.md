---

# Brane Developer Guide: Packaging & Containerizing Analytical Source Code

**Audience:** Software Engineers, Research Developers, and Tool Authors

**Purpose:** Learn how to wrap custom application logic (Python, R, binaries, or shell scripts) into portable, reusable Brane packages for execution within distributed workflows.

---

##  Critical Scope & Prerequisites

Before you begin developing packages, please review these architectural dependencies:

1. **Infrastructure Access:** This documentation assumes that a functional Brane infrastructure instance has already been deployed and configured for your environment. If you need access, credentials, or client certificates for an existing cluster, contact your **System Administrator**.

2. **Deploying Your Own Cluster:** If you are setting up your own control nodes or worker environments from scratch, the deployment process is *not* covered here. You must read the **Brane Installation Guide** to stand up the infrastructure before proceeding with this manual.

3. **Data Policies & Governance:** This guide focuses strictly on package development. It **does not cover data policy enforcement or governance rules**. If your workflow requires data compliance, residency constraints, or privacy policies, please read the dedicated **Data Policy Guide**.

4. **Execution Scope:** This documentation includes complete technical instructions to execute and test your code in two environments: **locally** on your development machine (Sandbox mode) and **remotely** on a deployed production infrastructure.


5. **Tooling Prerequisite:** You must have the **Brane CLI binary** installed on your local machine to interact with the environment, build packages, and run local simulation tests.

---

## 0. Installation & Setup

Before building packages, you must download the pre-compiled `brane` CLI binary directly from the official GitHub releases page and add it to your system path.
see instruction in the guide [installing brane tools](08-brane-tools.md)
 
---

## 1. Overview of the Packaging Concept

A **Brane Package** is an Executable Code Unit (ECU)—a portable, containerized computing unit that abstracts custom code into a language-agnostic function. When a package is successfully published, workflow authors can invoke its logic inside a BraneScript or Bakery pipeline without needing to understand its underlying codebase, internal language dependencies, or execution environment.

The developer package lifecycle follows five distinct phases:

1. **Initialize:** Generate the standard filesystem template via the CLI.

2. **Implement:** Write your raw functional logic in your preferred programming language.

3. **Manifest:** Create structural metadata (`container.yml`) detailing parameters, input constraints, and entry points.

4. **Compile:** Build the component locally into an isolated image using the runtime system.

5. **Publish:** Push the finished containerized asset to the cluster registry.

---

## 2. Package Architecture & The I/O Contract

Every compliant Brane software package must strictly implement the following filesystem layout:

```css
my_package/
├── container.yml      Required: Defines function schemas, parameters, and entry points
├── data.yml           Optional: Contains custom cross-package data type mappings
└─ code/              Required: Houses script sources, binaries, and internal dependencies
    ├── main.py        Executable code file
    └─ requirements.txt Language-specific dependency manifests

```

### The Architectural Communication Contract

Brane packages are expected to follow a strict, isolated I/O pattern to ensure language-agnostic execution across distributed worker nodes:

* **Input Payloads:** Brane constructs and delivers input arguments to your container at runtime as a structured YAML payload via an `INPUT` environment variable or dedicated configuration file. Your internal application logic must parse this YAML structure to extract its operational parameters.

* **Execution Outputs:** The containerized application must compile its results and stream a validated YAML string directly back to `stdout`.

* **File/Data Preservation:** Any persistent side-effects, large datasets, or final generated file outputs must be targeted to a designated workspace directory (commonly `/result`) for Brane to securely capture and persist.

---

## 3. Step-by-Step Implementation Workflow

### Step 1: Initialize the Template Workspace

Generate a validated filesystem structure using the user toolset:

```bash
brane new data_transformer
cd data_transformer

```

### Step 2: Authoring `container.yml`

The manifest explicitly registers your tool's signature with the orchestration layers. It describes core metadata, system-level dependencies (e.g., specific APT dependencies to run the environment), file paths, and runtime action signatures:

```yaml
name: data_transformer
version: 1.0.0
kind: ecu
description: "Executes target scaling and data normalization protocols."
owners:
  - "Your Name <you@example.org>"

image: "python:3.11-slim"

dependencies:
  ubuntu:
    - python3
    - python3-yaml

files:
  - source: "./code"
    target: "/opt/data_transformer/code"

env:
  - name: PYTHONUNBUFFERED
    value: "1"

install:
  - "pip install pyyaml"

entrypoint:
  kind: "task"
  exec: "python3 /opt/data_transformer/code/main.py"

```

### Step 3: Implementing the Code Pattern

Create your script file directly inside your dedicated source directory. This example utilizes the strict Brane YAML I/O environment contract:

```python
# code/main.py
import os
import sys
import yaml

def process_data(dataset_path, factor):
    # Core processing logic goes here
    return f"Processed {dataset_path} using factor {factor}"

def main():
    # 1. Safely extract input YAML from environment variable
    input_yaml = os.environ.get("INPUT", "{}")
    data = yaml.safe_load(input_yaml)

    # 2. Extract arguments mapped from the workflow interface
    path_arg = data.get("raw_dataset", "")
    factor_arg = float(data.get("scaling_factor", 1.0))
    
    # 3. Execute core logic
    result_string = process_data(path_arg, factor_arg)
    
    # 4. Stream mandated YAML structured response back to stdout
    output = {"result": result_string}
    yaml.safe_dump(output, sys.stdout)

    # 5. Persist heavy assets into the mandated workspace volume
    os.makedirs("/result", exist_ok=True)
    with open("/result/output.txt", "w") as f:
        f.write(result_string)

if __name__ == "__main__":
    main()

```

### Step 4: Compile the Execution Container

Run the build command to generate an isolated system image. The runtime will automatically analyze your codebase configurations, compile dependencies, validate metadata integrity, and store a tarball artifact within your local cache:

```bash
brane package build .

```

Verify your built image parameters using the inspection tool:

```bash
brane inspect data_transformer:1.0.0

```

---

## 4. Local vs. Remote Execution Modes

Understanding where and how your package executes is critical for troubleshooting and proper architectural planning.

### Local Execution (Development & Sandbox)

When you test a package using `brane package test` on your workstation, execution happens completely **locally**.

* **How it works:** The CLI interacts directly with your local container daemon, stands up a temporary sandbox, maps mock inputs, and streams standard output straight to your terminal screen.

* **Use case:** Fast debugging, checking internal dependencies, and running unit tests without affecting the shared cluster registry.

```bash
# Verify complex functions locally by passing explicit argument parameters
brane package test data_transformer:1.0.0 transform_metrics --input '{"raw_dataset": "/tmp/data.csv", "scaling_factor": 1.5}'

```

### Remote Execution (Production & Workflows)

Once a package is compiled and pushed to the cluster (`brane package push`), it enters **remote execution mode**.

* **How it works:** The package image is securely transmitted to the central registry. When a user triggers a workflow referencing your component, the central Control Node delegates the runtime block directly down to specific remote Worker Nodes where the target datasets physically reside.

* **Use case:** Live production analytical pipelines, multi-site computations, and processing secured datasets under strict institutional data policies.

```bash
brane package push data_transformer:1.0.0

```

---

## 5. Advanced Package Lifecycle Management

The Brane CLI provides dedicated workflows for ingesting, tracing, and safely deprecating local and remote package components.

### Importing and Pulling Existing Packages

Instead of authoring packages from scratch, you can pull production blueprints directly from shared remote endpoints or standard source control repositories:

```bash
# Import an entire package project directly from a Git source repository
brane package import https://github.com/your-org/your-package-repo.git

# Pull a verified, pre-built package down from your remote Brane instance registry
brane package pull data_transformer:1.0.0

```

### Package Cleanup

To free up local compute resources or decommission deprecated assets from centralized tracking indexes:

```bash
# Delete a package from your local development engine cache
brane package remove data_transformer:1.0.0

# Unpublish an asset from the remote cluster registry index (requires administrative elevation)
brane package unpublish data_transformer:1.0.0

```

---

## 6. Developer Reference Summary

| Lifecycle Phase | Target Command | Foundational Purpose |
| --- | --- | --- |
| **Initialize** | `brane new <name>` | Generates the structural skeleton directory.  |
| **Import** | `brane package import <url>` | Clones an asset pipeline from a remote git repository.  |
| **Pull** | `brane package pull <package>` | Fetches an operational package down from a remote instance registry.  |
| **Inspect** | `brane inspect <package>:<version>` | Validates structural inputs and metadata schemas.  |
| **Compile** | `brane package build .` | Builds the source codebase into an isolated, local container.  |
| **Test (Local)** | `brane package test <package> <func>` | Safely executes unit tests offline using local runtime mocks.  |
| **Publish (Remote)** | `brane package push <package>` | Commits and registers the finished image into the global cluster.  |
| **Inventory** | `brane package list` | Traces all currently cached local and active remote packages.  |

---

## 7. Troubleshooting & Verification Checklist

#### Compilation Failures

* Verify your manifest `image:` parameter points to a reachable, base container registry.

* Validate that your system dependencies listed under the `install:` or `dependencies:` headers are compatible with the base distribution layer.

* Ensure all files and directories mapped under the `files:` object physically exist inside your workspace.

#### Runtime and Crashes

* Capture exceptions within your scripts and route internal application crashes explicitly to `stderr`.

* Ensure your dictionary extractions cleanly fallback on default parameters if expected keys are absent from the parsed `INPUT` environment payload.

* Verify that your execution code explicitly initialises directory paths before writing data payloads to `/result` to avoid file descriptor permission errors.

#### Synchronization and Discovery Issues

* Ensure the target semantic `version:` string in your modified `container.yml` has been explicitly updated before issuing a new `push` command.

* Double-check your active client certificates and network endpoints if a newly published component fails to appear inside the shared registry.
