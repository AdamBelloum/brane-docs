# Brane Overview & Getting Started

_Last updated: October 2025_  
**Audience:** New users, educators, and evaluators  
**Prerequisites:** Basic familiarity with command-line tools and Docker  

---

## 1. What Is Brane?

**Brane** is a **programmable orchestration framework** that simplifies how data-intensive applications are executed across different computing environments â   from a single laptop to distributed infrastructures.

It helps **scientists, engineers, and system administrators** build, deploy, and execute workflows using lightweight, reusable building blocks called **Brane packages**.  
These packages wrap any code (Python, R, shell scripts, etc.) into portable, executable units.

###  Key Idea
Brane abstracts away technical complexity (containers, data transfers, security policies) so that users can:

- **Compose workflows** in an intuitive scripting language called **BraneScript**  
- **Package their code** once and run it anywhere  
- **Execute across multiple sites** securely and transparently  

###  Core Architecture (Simplified)

At its core, Brane consists of:

- **Control Node** â  central coordinator  
- **Worker Nodes** â   execute computations  
- **Proxy Nodes** *(optional)* â  bridge networks between sites  

All nodes communicate through secure channels and coordinate tasks via the Brane runtime.

---

## 2. Roles in Brane

Brane supports several types of users, each with distinct goals and tools.

| **Role** | **Description** | **Main Tool(s)** | **Typical Tasks** |
|-----------|-----------------|------------------|-------------------|
| **Scientist / End User** | Composes and runs workflows using existing functions | `brane` CLI | Write BraneScripts, run experiments |
| **Software Engineer / Developer** | Wraps existing code as reusable Brane packages | `brane` CLI | Create, build, and publish packages |
| **System Administrator** | Installs and manages Brane infrastructure | `branectl` CLI | Deploy and monitor nodes (control, worker, proxy) |

##  3. Key Components and Tools

### `brane`

The **user-facing command-line tool** for:

- Creating and building Brane packages  
- Running and testing workflows  
- Managing datasets and registries  

Example:

```bash
brane build
brane run helloworld
```

### `branectl`

The **administrative command-line tool** for:

* Installing and configuring nodes
* Generating configuration and certificate files
* Starting Brane services (control, worker, proxy)

Example:

```bash
branectl download services central -f
branectl start central
```

Together, these two tools allow users and administrators to operate the full Brane ecosystem.

---

## 4. Your First Workflow HelloWorld 

This short tutorial guides you through running your **first Brane workflow locally**.
It demonstrates the end-to-end flow: package creation workflow execution.

---

### Step 1. Verify Installation

Ensure Brane CLI is installed:

```bash
brane --version
```

If not installed, follow the instructions in the **[Installation & Setup Guide](installation.md)**.

---

### Step 2. Create a Simple Package

Initialize a new Brane package:

```bash
brane new helloworld
cd helloworld
```

Create a simple Python script:

```python
# helloworld.py
def hello_world():
    return "Hello, world!"
```

Edit the `container.yml` file to describe your package:

```yaml
name: hello_world
version: 1.0.0
functions:
  hello_world:
    command: python3 helloworld.py
```

---

### Step 3. Build and Register the Package

```bash
brane build
brane push
```

This compiles your code into a containerized Brane package and makes it available for use.

---

### Step 4. Create a Workflow Script

Create a new file called `HelloWorld.bs`:

```branescript
// HelloWorld.bs
// A simple workflow that prints "Hello, world!"

import hello_world;

println(hello_world());
```

---

### Step 5. Run the Workflow

Execute the workflow locally:

```bash
brane workflow run HelloWorld.bs
```

Expected output:

```
Hello, world!
```

You have successfully run your first Brane workflow!

---

### Step 6. Try the Interactive REPL (Optional)

Instead of writing a file, you can experiment interactively:

```bash
brane workflow repl
```

Then type:

```branescript
import hello_world;
println(hello_world());
```

---

## 5. What's Next?

Now that you have completed your first workflow, you can:

* **Create more complex packages** see [Developer Guide](developer-guide.md)
* **Compose multi-step workflows** see [BraneScript User Guide](branescript-guide.md)
* **Deploy Brane on multiple nodes** see [Installation & Setup Guide](installation.md)
* **Explore Brane's architecture** see [Conceptual & Architecture Guide](architecture.md)

---

## Summary

| You Learned                           | You Can Now                         |
| ------------------------------------- | ----------------------------------- |
| What Brane is and how its structured | Understand roles and components     |
| How to use the Brane CLI tools        | Install, build, and run packages    |
| How to run your first workflow        | Execute a HelloWorld in BraneScript |

---

**Brane empowers you to focus on science and data logic â  not on infrastructure details.**

Continue with the [Developer Guide](developer-guide.md)
or [? Back to Table of Contents](brane-docs-index.md)
---

TODO: generate a **front-matter block** (YAML metadata for docs frameworks like MkDocs or Docusaurus) so it can be integrated into documentation site navigation automatically?
