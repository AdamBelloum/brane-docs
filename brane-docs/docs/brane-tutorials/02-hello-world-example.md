# 3. Hello, World!


In this part, you:

- Install the Brane CLI client (`brane` executable),
- Write a simple Python function `hello_world()` returning `"Hello, world!"`,
- Describe it as a Brane package (`container.yml`),
- Build the package,
- Run it locally as a task and inside a workflow.

> **CLI note:** All `brane` commands shown here reflect the tutorial version (around Brane 2.0.0). The exact syntax may have changed; treat commands as **patterns** and update them to current CLI semantics later.

---

## 3.1. Background: workflows and tasks

Brane executes **workflows**: high‑level descriptions of algorithms or data processing pipelines.

- A workflow consists of **tasks** (conceptual functions with inputs and outputs).
- Tasks are composed with control flow (order, parallelism, branching).
- It helps to think of workflows as **graphs**:
  - Nodes: task calls.
  - Edges: data dependencies between tasks.

Example conceptual workflow:

- Run task $$f$$ on some input.
- In parallel, run another $$f$$ and $$h$$ using results of the first $$f$$.
- Then run $$g$$ on outputs from those tasks.

BraneScript is Brane’s **Domain‑Specific Language (DSL)** for describing workflows:

- Script-like syntax,
- Tasks look like functions,
- Control flow uses common constructs (if, for, while, `on`, `parallel`, etc.).

## 3.2. Objective

We implement a single task:

- Function name: `hello_world()`,
- Returns the string `"Hello, world!"`.

Plain Python logic:

```python
def hello_world():
    return "Hello, world!"
```
In Brane:

- We package this logic as a task in a container.
- We write a small BraneScript workflow that calls the task and prints the result.

## 3.3. Installing the Brane CLI
Download the brane executable from the repository or prebuilt binaries. Binaries typically follow the pattern:

```text
brane-<os>-<arch>
<os>: windows, darwin (macOS), linux
<arch>: x86_64 or aarch64 (M1/M2 Macs)
Rename the executable to brane for convenience:
```

```bash
# Windows
move .\brane-windows-x86_64 .\brane

# macOS (Intel)
mv ./brane-darwin-x86_64 ./brane

# macOS (M1/M2)
mv ./brane-darwin-aarch64 ./brane

# Linux
mv ./brane-linux-x86_64 ./brane
(Optional but recommended on Unix) Put brane in your PATH:
```

```bash
sudo mv ./brane /usr/local/bin/brane
Verify installation:

```bash
brane --version
You should not see “command not found” errors.
If you prefer not to modify PATH, you can invoke ./brane instead of brane and adjust later.
```

## 3.4. Writing the code (hello.py)
We implement the task logic in Python. Create a file hello.py:

```python
#!/usr/bin/env python3

def hello_world():
    return "Hello, world!"

print(f'output: "{hello_world()}"')
```
Explanation:

- #!/usr/bin/env python3
  - Shebang line. It tells the OS to run this file with python3 when you execute ./hello.py. Brane will call the script directly (as an executable), so this line is important.
- hello_world()
  - Returns "Hello, world!" – the core logic.
- print(f'output: "{hello_world()}"')
  - Prints the result to stdout using a simple YAML‑like convention:
- Prefix with output:,
  - Wrap the value in quotes.
  - Brane expects the program to print outputs in a predictable format so it can parse them as named results (here, a single named result output).

## 3.5. Writing container.yml (package manifest)
We now describe the package metadata and task interface in container.yml:

```yaml
# Generic package properties
name: hello_world
version: 1.0.0
kind: ecu

# Dependencies to install in the container
dependencies:
  - python3

# Files to include in the package
files:
  - hello.py

# Entrypoint: file to execute when running the package
entrypoint:
  kind: task
  exec: hello.py

# Tasks (actions) provided by this package
actions:
  hello_world:
    command:
    input:
    output:
      - name: output
        type: string
```

Key points:

Metadata:
- name: hello_world
- version: 1.0.0
- kind: ecu (Executable Code Unit).
Dependencies:
- python3 is installed inside the Ubuntu base image used by Brane.
Files:
- hello.py is included in the container.
- Paths are relative to the location of container.yml.
Entrypoint:
- Brane will execute hello.py directly inside the container.
- Shebang (#!/usr/bin/env python3) ensures the OS knows how to run it.
Actions (tasks):
- Defines a task hello_world:
- No input parameters (empty).
- One output named output of type string.
- The output name must match what the script prints (output: "...").
- In pseudo‑signature form:

```text
hello_world() -> string
```
## 3.6. Building the package
Brane uses Docker and Buildx to build package containers. Make sure:

Docker is installed (Docker Desktop for Windows/macOS, Docker engine for Linux).
Buildx is available (ships with recent Docker; older setups may require manual install).
Then build the package:

```bash
# Syntax to be validated later
brane build ./container.yml
```

You should see a success message similar to:

```text
Successfully built version 1.0.0 of container (ECU) package hello_world.
```
List local packages:

```bash
# Syntax to be validated later
brane list
```

You should see hello_world among the packages.

## 3.7. Running the package in the test environment
The brane test subcommand lets you:

Select a task from a package,
Provide inputs,
See outputs.
Run:

```bash
# Syntax to be validated later
brane test hello_world
```
You’ll see a prompt to choose the function (only hello_world in this package). Since there are no inputs, Brane will:

Execute the container,
Show the result, e.g.:
```text
Result: Hello, world! [String]
```
Note:

The first run may be slower (Docker pulls and loads the image).
Subsequent runs are faster due to caching.

## 3.8. Running a local workflow (BraneScript)
Now write a simple workflow that calls the task and prints the result.

Create workflow.bs:

```branescript
import hello_world;
println(hello_world());
```
Explanation:

-  import hello_world;
  - Makes the package available in the workflow. Omitting a version lets Brane pick the latest.
- println(hello_world());
  -  Calls the hello_world task and passes its result to a built‑in println() function that prints it to stdout.

Run:

```bash
# Syntax to be validated later
brane run ./workflow.bs
```
You should see:

```text
Hello, world!
```
## 3.9. Using the REPL (interactive mode)
Brane provides a REPL (Read‑Eval‑Print Loop) for BraneScript:

```bash
# Syntax to be validated later
brane repl
```

Example interactive session:

```text
Welcome to the Brane REPL, press Ctrl+D to exit.

1> import hello_world;
2> println(hello_world());
Hello, world!
3> for (let i := 0; i < 3; i := i + 1) { println(hello_world()); }
Hello, world!
Hello, world!
Hello, world!
4> _
```
This is useful for quick experiments and debugging.

## 3.10. Summary
In Part 1 you:

- Installed the Brane CLI,
- Wrote a minimal Python function and wrapped it with a simple output convention,
- Defined container.yml to describe the package,
- Built and tested the package locally,
- Wrote and ran a small BraneScript workflow calling the hello_world task.

Part 2 will extend this exercise to a more complex pipeline (Disaster Tweets), showing how Brane handles datasets and multi‑step workflows.