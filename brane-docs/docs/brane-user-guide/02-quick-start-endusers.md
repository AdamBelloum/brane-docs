## 2. Quick Start for Scientists

This section gives scientists a pragmatic path to go from zero to a running workflow. Details about installation and deployment are covered later for system engineers.

### 2.1. Installing the Brane CLI

Brane offers a command-line client (Brane CLI) for interacting with a Brane instance:

- Submitting workflows,
- Managing local packages,
- Interacting with remote registries.

> **Note**: CLI commands shown here are conceptual; exact syntax may differ in your installation. We will validate and update command examples later.

Typical steps:

1. Install **Docker** and **Buildx** (if required by your Brane distribution).
2. Install the Brane client (e.g., via a package, script, or binaries provided by your deployment).
3. Confirm the CLI is available:

- For example, run a help command to list available subcommands (syntax to be validated).

### 2.2. Connecting to a Brane instance

Before running workflows, the CLI needs to know which Brane instance to use and how to authenticate.

Conceptually, you will:

**Configure the instance endpoint**  

- Specify the URL or hostname of the Brane control node’s proxy.

**Provide client credentials**  

- Install client certificates or tokens that identify you to the instance.
- These may be generated and distributed by an administrator or system engineer.

Once configured, you can:

- List remote packages,
- Submit workflows,
- Access datasets hosted by the instance (subject to policies).

### 2.3. Running a simple workflow

With the CLI configured:

**Write a small BraneScript or Bakery workflow**  

- Import a simple package (e.g., a hello-world or base64 example).
- Call one of its functions.
- Print or commit results.

**Submit the workflow**  

- Use the CLI to send the workflow to the Brane instance.
- The instance’s driver and planner will:
- Validate the workflow,
- Plan tasks across domains,
- Execute them via worker nodes.

**Inspect results**  

- View logs or output provided by the CLI.
- Access any datasets or intermediate results produced.

### 2.4. Where to go next

After a successful quick start:

- To **build your own packages**, see **[Section 4](04-software-engineers.md)**.
- To **work with datasets** more seriously, see **[Section 5](05-data-policy-experts.md)**.
- To understand **policies**, see **[Section 5](05-data-policy-experts.md)**.
- For deeper infrastructure details, see **[Section 6](06-administrators.md)**.

---
