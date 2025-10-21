---
````markdown
# Tutorials & Examples

_Last updated: October 2025_  
**Audience:** Beginners, educators, new contributors  
**Purpose:** Guided, hands-on examples to learn Brane through practical exercises.  
---

## Overview

This section contains short, progressive tutorials designed to help you get comfortable with Brane.  
You will learn how to:

1. Run a simple **HelloWorld** workflow locally  
2. Deploy Brane on a small cluster  
3. Create and register your own Brane function  
4. Compose a multi-step workflow using **BraneScript**  
5. *(Preview)* Add a data policy with **eFLINT**  

Each tutorial assumes you have installed Brane using the [Installation & Setup Guide](installation.md).

---

## Tutorial 1 - Run HelloWorld Locally

### Objective
Run your first Brane workflow on a single machine.

### Prerequisites
- Brane installed and running locally (`branectl start central`)
- Docker running
- `brane` CLI accessible from your terminal

---

### Step 1 - Verify Setup
```bash
brane --version
````

Check that the Brane API is active:

```bash
docker ps | grep brane-api
```

---

### Step 2 - Create a Simple Package

```bash
brane new helloworld
cd helloworld
```

Edit `code/main.py`:

```python
def say_hello():
    print("Hello, world!")
```

Update `container.yml`:

```yaml
name: hello_world
version: 1.0.0
language: python
functions:
  say_hello:
    command: python3 code/main.py
```

---

### Step 3  - Build and Push

```bash
brane build
brane push
```

---

### Step 4 - Run a Workflow

Create `HelloWorld.bs`:

```branescript
import hello_world;
println(hello_world::say_hello());
```

Run it:

```bash
brane workflow run HelloWorld.bs
```

Output:

```
Hello, world!
```

Congratulations - you’ve run your first Brane workflow locally!

---

## Tutorial 2 - Deploy Brane on a Small Cluster

### Objective

Set up Brane on multiple machines and run a distributed workflow.

### Prerequisites

* Two or more machines with Docker and `branectl` installed
* SSH access between nodes
* Basic understanding of YAML configuration files

---

### Step 1 - Define the Infrastructure

On the control node:

```bash
branectl generate infra -f -p ./config/infra.yml control:control.local worker:worker.local
```

---

### Step 2 - Generate Certificates

On each node:

```bash
branectl generate certs -f -p ./config/certs server <LOCATION_ID> -H <HOSTNAME>
```

Example (worker node):

```bash
branectl generate certs -f -p ./config/certs server worker -H worker.local
```

Copy each node’s `ca.pem` to the control node under:

```
config/certs/<node>/ca.pem
```

---

### Step 3 - Configure Nodes

**Control Node:**

```bash
branectl generate node -f central control.local
branectl start central
```

**Worker Node:**

```bash
branectl generate node -f worker control.local worker
branectl start worker
```

---

### Step 4 - Verify Cluster

On the control node:

```bash
brane instance list
```

Expected output:

```
NAME        TYPE      STATUS
control     control   ACTIVE
worker      worker    ACTIVE
```

Now you can run the HelloWorld workflow remotely - Brane will dispatch it to your worker node automatically.

---

## Tutorial 3 - Create and Register Your Own Function

### Objective

Wrap your own code as a Brane function and use it in workflows.

---

### Step 1 -  Initialize

```bash
brane new greetings
cd greetings
```

### Step 2 - Add Function Code

```python
# code/greet.py
import sys

def greet(name):
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet(sys.argv[1]))
```

---

### Step 3 - Define Metadata

Edit `container.yml`:

```yaml
name: greetings
version: 1.0.0
language: python
functions:
  greet:
    command: python3 code/greet.py
    inputs:
      name: string
    output: string
```

---

### Step 4 - Build and Push

```bash
brane build
brane push
```

---

### Step 5  -  Use It in BraneScript

```branescript
import greetings;

msg = greetings::greet("Ada");
println(msg);
```

Run it:

```bash
brane workflow run greet_test.bs
```

Output:

```
Hello, Ada!
```

---

## Tutorial 4 - Compose a Multi-Step Workflow in BraneScript

### Objective

Combine multiple packages in a workflow.

---

### Example Scenario

We’ll chain two operations:

1. Generate a message with `greetings`
2. Convert it to uppercase with `text_utils`

---

### Step 1 - Create the Script

```branescript
import greetings;
import text_utils;

msg = greetings::greet("Marie");
upper = text_utils::to_upper(msg);

println(upper);
```

---

### Step 2 - Run It

```bash
brane workflow run multi_step.bs
```

Expected output:

```
HELLO, MARIE!
```

---

### Step 3 — Visualize Workflow Execution

List executed workflows:

```bash
brane workflow list
```

Inspect logs for details:

```bash
brane workflow logs <workflow_id>
```

---

##  Tutorial 5  -  Add a Data Policy using eFLINT *(Coming Soon)*

### Objective

Learn how to attach and enforce data-sharing policies to Brane workflows.

---

### Planned Content

* Define a policy in **eFLINT** format
* Attach it to a workflow or dataset
* Control where data can be processed
* Monitor compliance in distributed runs

> This feature will be integrated in future versions of Brane’s policy framework.
> Check for updates in the [Release Notes](release-notes.md).

---

##Summary

| Tutorial                     | Key Skill Gained                  |
| ---------------------------- | --------------------------------- |
| 1. Run HelloWorld Locally    | Install and test Brane            |
| 2. Deploy Small Cluster      | Set up distributed Brane          |
| 3. Create Your Own Function  | Wrap custom code as a package     |
| 4. Multi-Step Workflow       | Compose packages in BraneScript   |
| 5. Add Data Policy (Preview) | Secure data execution with eFLINT |

---

### Next Steps

* Explore [BraneScript User Guide](branescript-guide.md) to deepen your scripting knowledge.
* Review [Administration & Operations Guide](admin-guide.md) for managing deployments.
* Visit [Configuration & Reference Manual](reference-manual.md) for detailed CLI and YAML options.

[? Back to Table of Contents](Brane-documentation-index.md)

```
