# Brane End-User & Workflow Authoring Guide

**Audience:** Data Scientists, Analysts, and Workflow Developers  
**Purpose:** Introduction to the Brane ecosystem, executing introductory tutorials, and authoring complex distributed workflows using BraneScript and Bakery.

---

## 1. Ground Rules & Operational Assumptions

Before you begin interacting with the framework, ensure you understand the boundaries of your environment and user role. This guide operates under three core assumptions:

1.  **Deployed Infrastructure:** We assume a functional Brane infrastructure cluster has already been successfully deployed by your infrastructure team. If you do not know the connection endpoints, domain names, or network parameters, **you must contact your system administrator** before attempting remote execution.
2.  **The Scope of Local Examples:** The `helloworld` project outlined in Section 4 is a simplified, single-node toy example designed solely to validate that your local CLI installation works. It does not reflect a full distributed production workflow.
3.  **Package Dependency & Collaboration:** To author meaningful, real-world workflows in BraneScript or Bakery, you must import pre-existing functional packages. If you cannot find the specialized tools, algorithms, or libraries required for your data analysis, **you should consult your software development team or system administrator** to verify if those packages have been built and pushed to the registry.

---

## 2. What Is Brane?

Brane is a **programmable orchestration framework** designed to simplify the execution of data-intensive applications across distributed computing environments—ranging from a local machine to multi-site infrastructures.

The framework abstracts away lower-level technical complexities such as container runtime management, cross-site data transfers, and security policies. This allows you to focus exclusively on your analytical logic:
*   **Compose workflows** using an intuitive, high-level scripting language (BraneScript) or declarative configurations (Bakery[g].
*   **Leverage reusable building blocks** (Brane packages) that wrap code written in Python, R, or standard shell scripts.
*   **Execute safely across domains** while maintaining full compliance with localized data governance policies[g].

### Core Roles in the Ecosystem

| Role | Core Objective | Primary Interfaces |
|---|---|---|
| **Scientist / End User** | Composes, executes, and monitors data workflows using existing functions. | `brane` CLI |
| **Software Developer** | Wraps raw source code into portable, reusable Brane packages. | `brane` CLI |
| **System Administrator** | Deploys, configures, and monitors the physical cluster nodes. | `branectl` CLI |

---

## 3. Setting Up Your Workspace

This guide assumes your infrastructure administrator has already deployed the Brane cluster, provisioned your user account, and provided you with your unique **client authentication certificates**.

### The `brane` User CLI
As an end-user, your primary tool is the `brane` CLI. It is entirely separate from the administrative `branectl` tool used by platform engineers. You will use `brane` to build packages locally, query data registries, and submit workflows.

Verify that your workstation has the tool installed correctly:
```bash
brane --version

```

*(If the command is not recognized, install brace CLI as described in the guide [intalling-brane-tools](08-brane-tools.md)*

---

## 4. Quick-Start Tutorial: Your First Workflow

This short walkthrough demonstrates the complete lifecycle of creating a local script package and executing your first workflow file.

### Step 1: Initialize a Project Workspace

Create a clean directory to store your package configurations:

```bash
brane new helloworld
cd helloworld

```

### Step 2: Write the Execution Script

Create a simple script named `helloworld.py` containing the logic you want to containerize:

```python
# helloworld.py
def hello_world():
    return "Hello, world!"

```

### Step 3: Describe the Package Interface

Every package requires a `container.yml` file. This metadata file acts as a manifest that maps your underlying code functions to the Brane runtime ecosystem:

```yaml
name: hello_world
version: 1.0.0
functions:
  hello_world:
    command: python3 helloworld.py

```

### Step 4: Build and Register

Compile your script and manifest into a containerized Brane package, making it available to the runtime environment:

```bash
brane build
brane push

```

### Step 5: Write and Execute the BraneScript Workflow

Create a new file named `HelloWorld.bs` in your workspace:

```branescript
// HelloWorld.bs
import hello_world;

println(hello_world());

```

Run the workflow locally using the CLI:

```bash
brane workflow run HelloWorld.bs

```

**Expected Output:**

```text
Hello, world!

```

---

## 5. Workflow Formulation Mechanics

Brane provides two distinct paradigms for authoring workflows: **BraneScript** (an imperative scripting language) and **Bakery** (a declarative configuration structure). Both compile down to an identical internal Workflow Intermediate Representation (WIR).

### 5.1 Imperative Formulation: BraneScript

BraneScript provides a structured environment featuring variables, strongly-typed functions, loops, and traditional control flow:

```branescript
import "text" version "1.0.0";

fn main() {
    let message: string = "Hello, Brane!";
    let result: string = text.uppercase(message);
    println(result);
}

```

### 5.2 Declarative Formulation: Bakery

Bakery allows authors to outline workflows using structured YAML syntax. This is ideal for straightforward pipeline definitions where data moves predictably from step to step:

```yaml
name: hello-workflow
steps:
  - id: uppercase
    package: text:1.0.0
    function: uppercase
    input:
      text: "Hello, Brane!"
    output:
      as: upper_text

  - id: print
    package: util:1.0.0
    function: println
    input:
      text: "{{ upper_text }}"

```

---

## 6. Execution Modes: Local vs. Remote

Brane supports two distinct execution environments depending on whether you are verifying syntax or running production analysis over live distributed datasets.

### 6.1 Local Execution Mode

* **Purpose:** For local validation, rapid prototyping, and workflow syntax testing.


* **Behavior:** The workflow runs entirely inside your local workstation environment using dummy configurations or local package images. It does not touch the production cluster network or trigger secure data movements.


* **Command:**
```bash
brane workflow run <WORKFLOW_FILE>

```



### 6.2 Remote Execution Mode

* **Purpose:** Production data analysis.


* **Behavior:** The CLI compiles your BraneScript/Bakery into WIR and submits it to the remote central infrastructure node. The cluster orchestrator verifies domain compliance rules, plans computation tasks across physical worker nodes, handles cross-site data transfers, and triggers secure computation.


* **Command Template:**
```bash
# Run a workflow on a specifically configured remote instance
brane workflow run <WORKFLOW_FILE> --remote <CENTRAL_PROXY_URL> --cert ./config/certs/client.pem --key ./config/certs/client-key.pem

```


* **Real-World Example:**
```bash
brane workflow run analysis_script.bs --remote 10.0.0.10 --cert ./config/certs/client.pem --key ./config/certs/client-key.pem

```



> 💡 **Tip:** If you have already paired your active environment using `brane instance use <INSTANCE_NAME>`, you do not need to pass the `--remote`, `--cert`, and `--key` flags explicitly every time you submit a job.
> 
> 

---

## 7. Managing Remote Assets: Packages & Datasets

Workflows rely on two foundational components: functional code units (packages) and persistent data sources (datasets).

### 7.1 Package Discovery

To see what packages have been approved and published to the active Brane instance registry by the engineering team, execute:

```bash
brane package list

```

To inspect the structural inputs, expected parameter types, and return values of a specific package function, use the information command:

```bash
brane package info <PACKAGE_NAME>:<VERSION>

```

### 7.2 Incorporating Datasets

Brane differentiates between persistent datasets (managed via explicit metadata) and transient intermediate results generated dynamically mid-workflow.

To scan the available active datasets you have permission to access:

```bash
brane data list

```

#### Utilizing Datasets in BraneScript

Reference your target dataset by its unique name identifier, and explicitly commit your analytical results back to the registry at the conclusion of your script:

```branescript
import "analysis" version "1.0.0";

fn main() {
    let ds = dataset("clinical_patients");   
    let summary = analysis.summarize(ds);

    commit(summary, name = "patients_summary_output");
}

```

---

## 8. Performance & Monitoring

When executing workflows in remote mode, you can monitor execution traces and pull execution history directly from the cluster orchestrator.

### Monitoring Background Jobs

For long-running distributed compute jobs, you can track performance metrics, trace execution status, and extract real-time operational logs using the unique identifier returned at submission:

```bash
brane workflow status <WORKFLOW_ID>
brane workflow logs <WORKFLOW_ID>

```

---

## 9. Troubleshooting Technical Faults

| Symptom | Probable Cause | Corrective Action |
| --- | --- | --- |
| `Package not found` | The package has not been pushed to the remote instance registry or contains a typo.

 | Run `brane package list` to confirm the precise spelling and version tags available on the cluster.

 |
| `Access denied / Data source missing` | The data governance policies of the hosting domain block your user certificate from reading the data.

 | Run `brane data list` to verify visibility. Contact the domain data steward or security expert to adjust authorization policies.

 |
| `Workflow execution failure` | Runtime exception thrown inside a containerized package function.

 | Stream execution details using `brane workflow logs <WORKFLOW_ID>`. Validate that all variable formats match the strict inputs defined in the package interface.

 |

---

## 📖 Deep-Dive Reference Manuals

For granular details on advanced development workflows and comprehensive scripting features, consult the complete specification guides:

* **For comprehensive scripting syntax, control flows, and standard library APIs:** See the [Official BraneScript Reference Guide](https://www.google.com/search?q=branescript-guide.md).
* **For package compilation standards, multi-stage compilation specifications, and container recipes:** See the [Official Brane Package Deployment Guide](https://www.google.com/search?q=developer-guide.md).

```

```
