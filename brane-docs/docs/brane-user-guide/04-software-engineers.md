## 4. Software Engineers: Developing Packages

This section is for **software engineers** who want to create and maintain Brane **packages** (Executable Code Units, ECUs).

You will learn how to:

- Describe a package with `container.yml`,
- Build and test a package locally,
- Publish a package to a Brane instance,
- Manage local and remote packages. 

> **Note:** CLI commands in this section follow the structure presented in the current manual (e.g., `brane package ...`). Exact syntax and flags may have changed; we will validate and update them later.

---

### 4.1. What is a Brane package (ECU)?

A Brane **package** is a containerized piece of code that Brane workflows can call as a function.]

Key properties:

- The package is built into a container image (e.g., Docker image).
- `container.yml` describes:
  - Package metadata (name, version, kind),
  - Dependencies and environment,
  - Which files to include,
  - How to run the package (entrypoint),
  - Which functions (actions) are exposed to workflows. 
- At runtime, Brane:
  - Starts the container on a worker node,
  - Passes input to the container (via environment variables and/or files),
  - Reads outputs (from stdout as YAML and from file paths such as `/result`).

Packages can be:

- **Local** – present only on your machine (built from your source tree),
- **Remote** – stored in a Brane instance’s registry. 

---

### 4.2. Authoring `container.yml`

`container.yml` is the package’s “manifest”. It tells Brane how to build and run your code.]

A typical `container.yml` contains:

- Basic metadata,
- Build steps and dependencies,
- File mappings,
- Runtime entrypoint and actions.

#### 4.2.1. Basic structure

Core fields (conceptual example):

```yaml
name: my_package
version: 1.0.0
kind: ecu

description: "Example package that demonstrates inputs and outputs."
owners:
  - "Your Name <you@example.org>"
name – unique package name.
version – version string.
kind – for code packages, typically ecu.
description / owners – metadata for humans and registries.
```

####  4.2.2. Environment and dependencies
Describe the runtime environment for your package:

```yaml
image: "debian:stable"

dependencies:
  ubuntu:
    - python3
    - python3-yaml

env:
  - name: PYTHONUNBUFFERED
    value: "1"
```
- image – base container image (for building/running).
-  dependencies – system-level packages (e.g., python3, python3-yaml) to install.
- env – environment variables inside the container.

Brane uses these fields to build the ECU image so that all required dependencies are present at runtime.

#### 4.2.3. Files and install steps
You specify which files to include and any install steps needed:

```yaml
files:
  - source: "./src"
    target: "/opt/my_package/src"

install:
  - "apt-get update"
  - "apt-get install -y python3 python3-yaml"
```
- files – copy code and resources into the container.
- install – commands run inside the container during build.
- Some packages also use postinstall for final configuration.

#### 4.2.4. Entrypoint and actions
The entrypoint defines how Brane runs your package:

```yaml
entrypoint:
  kind: "task"
  exec: "python3 /opt/my_package/src/main.py"
```
Actions (functions) are then described (conceptually) to map between Brane workflows and command-line invocation. The manual shows examples where:

- Inputs are passed as YAML via an INPUT environment variable or file,
- Outputs are written as YAML to stdout,
- Optional result files are written to /result for Brane to capture.

The precise schema for actions will follow from your current Brane version (e.g., how you declare function names, arguments, and result types); we will align this later when updating CLI and manifest details.

### 4.3. Building and testing packages locally
Before publishing, you typically:

1. Create container.yml and code files.
2. Build the package image.
3. Test the package locally using the Brane CLI.

#### 4.3.1. Building the package
Conceptually, you use a CLI command like:

```bash
# Syntax to be validated later
brane package build .
```
This command:

- Reads container.yml,
- Builds the container image (using Docker + Buildx),
- Registers the package in your local Brane client environment.


#### 4.3.2. Testing locally
To test:

1. Use the CLI to run the package function directly (e.g., a “hello world” or “base64 encode” function).

1. Provide input in the expected format (typically via arguments or via a YAML INPUT structure).

1. Inspect:
-  Standard output (YAML response),
- Any files written to /result inside the container.


Example pattern (conceptual):

```bash
# Syntax to be validated later
brane package test  my_package:1.0.0 my_function --input '...'
```
Internally, Brane:

- Constructs an INPUT payload according to the function signature,
- Starts the container,
- Reads the YAML output and any files under /result.

Testing this locally before publishing ensures that:

- Dependencies are correct,
- Inputs and outputs are handled as expected,
- Container startup and execution behave properly.

### 4.4. Publishing packages to a Brane instance
Once the package is working locally, you can publish it to a Brane instance so others can use it.


Typical steps:

1. Log in / authenticate to a Brane instance:
- Configure instance endpoint and client certificates (see Admin section).
1. - Push the package to the instance:
Use a CLI command to upload the image and metadata.
1. Verify that the package appears in the instance’s package list.

#### 4.4.1. Pushing a package
Conceptual CLI workflow (syntax to be validated):

```bash
# Authenticate to the instance first (details depend on your setup)
brane package push my_package:1.0.0
```
This operation:

- Uploads the built ECU image and metadata,
- Registers the package in the instance’s central registry.

After pushing:

- Scientists and other users can discover and use the package in their workflows,
-  You can update the package later by pushing a new version (e.g., 1.1.0).

#### 4.4.2. Verifying published packages
You (or users) can list remote packages:

```bash
# List locally cached packages
brane package list
```
This shows:

- Package names and versions,
- Whether they are local or remote,
-  Basic metadata (description, owners).

### 4.5. Managing local and remote packages
The Brane CLI distinguishes between local and remote packages.
 
- Local packages:
- Built on your machine from your source code,
- Useful for development and local testing.
- Remote packages:
- Hosted on a Brane instance’s registry,
- Available to anyone with access and permissions.

#### 4.5.1. Importing packages from Git or remote sources
The manual describes importing packages from Git repositories or remote instances:
 
- You can import a package from a GitHub repo into your local environment.
- You can pull a remote package from a Brane instance to get a local copy.
- Conceptual commands (syntax to be validated):

```bash
# Import from GitHub
brane package import https://github.com/your-org/your-package-repo.git

# Pull a remote package from an instance
brane package pull some_package:1.2.3
```
This allows you to:

- Start from existing packages,
- Inspect or modify their code locally,
- Rebuild and publish customized versions.

#### 4.5.2. Removing packages
You can clean up packages that are no longer needed.

Conceptual commands (syntax to be validated):

```bash
# Remove a local package
brane package remove my_package:1.0.0

# Unpublish a package from a remote instance
brane package unpublish my_package:1.0.0
```
These operations:

Free up local resources (images, metadata),
Remove remote packages that should no longer be used (subject to appropriate admin rights).

#### 4.5.3. Best practices for package developers
- Use semantic versioning (e.g., 1.0.0, 1.1.0) to make upgrades explicit.
- Keep container.yml in version control along with your code.
- Test locally before publishing:
- Verify inputs/outputs,
- Check performance and error handling.
- Document:
- Expected inputs and outputs,
- Dataset requirements,
- Any side-effects (e.g., external services used).


### 4.6. Summary
For software engineers, Brane provides:

- A package model (ECUs) where you package your logic as containers.
- A manifest (container.yml) describing metadata, dependencies, files, entrypoint, and actions.
- CLI workflows to:
- Build and test packages locally,
- Publish packages to a Brane instance,
- Import, pull, and remove packages from various sources.
 
These packages become the building blocks that scientists use in their workflows (see Section 6), and their correct definition and behaviour are essential for reliable, policy-aware execution in Brane.


### 4.7. Developer Quick Reference

This section summarizes the minimum you need to develop and test a Brane package.

---

#### 4.7.1. Minimal `container.yml` template (Python example)

```yaml
name: my_package
version: 0.1.0
kind: ecu

description: "Minimal example Brane package."
owners:
  - "Your Name <you@example.org>"

image: "python:3.11-slim"

dependencies:
  ubuntu:
    - python3
    - python3-yaml

files:
  - source: "./src"
    target: "/opt/my_package/src"

env:
  - name: PYTHONUNBUFFERED
    value: "1"

install:
  - "pip install pyyaml"

entrypoint:
  kind: "task"
  exec: "python3 /opt/my_package/src/main.py"
```

Directory structure:

```text
my-package/
  container.yml
  src/
    main.py
```
#### 4.7.2. I/O contract for package code (conceptual)
Brane packages are expected to follow a simple I/O pattern:

Input:
-  Provided as a YAML structure (e.g., via an INPUT environment variable or file).
  -  Your code parses this YAML to get function arguments.
-  Output:
  -  Write a YAML response to stdout.
  -  Optionally write a result file or directory to a known path (commonly /result) for Brane to capture.

Example main.py (conceptual):

```python
import os
import sys
import yaml

def main():
    # Read input YAML from environment variable (pattern may vary by Brane version)
    input_yaml = os.environ.get("INPUT", "{}")
    data = yaml.safe_load(input_yaml)

    # Extract arguments
    text = data.get("text", "")
    times = int(data.get("times", 1))

    # Do work
    result = text * times

    # Prepare output as YAML
    output = {"result": result}

    # Write YAML to stdout
    yaml.safe_dump(output, sys.stdout)

    # Optionally write a file result (Brane can capture /result or similar)
    os.makedirs("/result", exist_ok=True)
    with open("/result/output.txt", "w") as f:
        f.write(result)

if __name__ == "__main__":
    main()
```

Note: The exact input/output conventions (e.g., which env vars or paths to use) may depend on the current Brane version and its runtime contract. Adjust according to your deployment’s documentation.

#### 4.7.3. Typical lifecycle (commands to be validated)
1. Build locally

```bash
# In the package directory with container.yml
brane package build .
```
2. Test locally

```bash
# Test a function exposed by the package
brane package test my_package:0.1.0 my_function \
    --input '{"text": "Hello", "times": 3}'
```
Check:

-  stdout (YAML output),
-  any files under /result inside the container.

3. Publish to an instance

```bash
# Assuming instance and credentials are configured
brane package push my_package:0.1.0
```

4. List packages (local/remote)

```bash
brane package list
```
All command examples above are structural and may need updating to match the current brane CLI. Use them as patterns, not exact syntax, until we revise them with up‑to‑date notes.

#### 4.7.4. Troubleshooting checklist
Build fails:
-  Check base image and dependencies/install commands.
-  Ensure paths in files exist and are correct.
-  Package runs but crashes:
-  Log any exceptions and print them to stderr.
-  Verify that input YAML contains the keys you expect.
-  Check that your code can write to /result (permissions, paths).
-  Published package not visible:
-  Confirm you are connected to the correct instance.
-  Check registry health on the Brane instance.
-  Ensure version (version in container.yml) is unique and correctly specified.
-  These patterns cover the common path from “new package skeleton” to “published, usable component” in Brane workflows.





## 4. Software Engineers: Developing Packages

This section is for **software engineers** who want to create and maintain Brane **packages** (Executable Code Units, ECUs).

You will learn how to:

- Describe a package with `container.yml`,
- Build and test a package locally,
- Publish a package to a Brane instance,
- Manage local and remote packages. [1]

> **Note:** CLI commands in this section follow the structure presented in the current manual (e.g., `brane package ...`). Exact syntax and flags may have changed; we will validate and update them later.

---

### 4.1. What is a Brane package (ECU)?

A Brane **package** is a containerized piece of code that Brane workflows can call as a function. 

Key properties:

- The package is built into a container image (e.g., Docker image).
- `container.yml` describes:
  - Package metadata (name, version, kind),
  - Dependencies and environment,
  - Which files to include,
  - How to run the package (entrypoint),
  - Which functions (actions) are exposed to workflows. 
- At runtime, Brane:
  - Starts the container on a worker node,
  - Passes input to the container (via environment variables and/or files),
  - Reads outputs (from stdout as YAML and from file paths such as `/result`).

Packages can be:

- **Local** – present only on your machine (built from your source tree),
- **Remote** – stored in a Brane instance’s registry. 

---

### 4.2. Authoring `container.yml`

`container.yml` is the package’s “manifest”. It tells Brane how to build and run your code.

A typical `container.yml` contains:

- Basic metadata,
- Build steps and dependencies,
- File mappings,
- Runtime entrypoint and actions.

#### 4.2.1. Basic structure

Core fields (conceptual example):

```yaml
name: my_package
version: 1.0.0
kind: ecu

description: "Example package that demonstrates inputs and outputs."
owners:
  - "Your Name <you@example.org>"
name – unique package name.
version – version string.
kind – for code packages, typically ecu.
description / owners – metadata for humans and registries.
```

####  4.2.2. Environment and dependencies
Describe the runtime environment for your package:

```yaml
image: "debian:stable"

dependencies:
  ubuntu:
    - python3
    - python3-yaml

env:
  - name: PYTHONUNBUFFERED
    value: "1"
```
- image – base container image (for building/running).
-  dependencies – system-level packages (e.g., python3, python3-yaml) to install.
- env – environment variables inside the container.

Brane uses these fields to build the ECU image so that all required dependencies are present at runtime.

#### 4.2.3. Files and install steps
You specify which files to include and any install steps needed:

```yaml
files:
  - source: "./src"
    target: "/opt/my_package/src"

install:
  - "apt-get update"
  - "apt-get install -y python3 python3-yaml"
```
- files – copy code and resources into the container.
- install – commands run inside the container during build.
- Some packages also use postinstall for final configuration.

#### 4.2.4. Entrypoint and actions
The entrypoint defines how Brane runs your package:

```yaml
entrypoint:
  kind: "task"
  exec: "python3 /opt/my_package/src/main.py"
```
Actions (functions) are then described (conceptually) to map between Brane workflows and command-line invocation. The manual shows examples where:

- Inputs are passed as YAML via an INPUT environment variable or file,
- Outputs are written as YAML to stdout,
- Optional result files are written to /result for Brane to capture.

The precise schema for actions will follow from your current Brane version (e.g., how you declare function names, arguments, and result types); we will align this later when updating CLI and manifest details.

### 4.3. Building and testing packages locally
Before publishing, you typically:

1. Create container.yml and code files.
2. Build the package image.
3. Test the package locally using the Brane CLI.

#### 4.3.1. Building the package
Conceptually, you use a CLI command like:

```bash
# Syntax to be validated later
brane package build .
```
This command:

- Reads container.yml,
- Builds the container image (using Docker + Buildx),
- Registers the package in your local Brane client environment.


#### 4.3.2. Testing locally
To test:

1. Use the CLI to test the package function directly (e.g., a “hello world” or “base64 encode” function).

1. Provide input in the expected format (typically via arguments or via a YAML INPUT structure).

1. Inspect:
-  Standard output (YAML response),
- Any files written to /result inside the container.


Example pattern (conceptual):

```bash
# Syntax to be validated later
brane package test  my_package:1.0.0 my_function --input '...'
```
Internally, Brane:

- Constructs an INPUT payload according to the function signature,
- Starts the container,
- Reads the YAML output and any files under /result.

Testing this locally before publishing ensures that:

- Dependencies are correct,
- Inputs and outputs are handled as expected,
- Container startup and execution behave properly.

### 4.4. Publishing packages to a Brane instance
Once the package is working locally, you can publish it to a Brane instance so others can use it.


Typical steps:

1. Log in / authenticate to a Brane instance:
- Configure instance endpoint and client certificates (see Admin section).
1. - Push the package to the instance:
Use a CLI command to upload the image and metadata.
1. Verify that the package appears in the instance’s package list.

#### 4.4.1. Pushing a package
Conceptual CLI workflow (syntax to be validated):

```bash
# Authenticate to the instance first (details depend on your setup)
brane package push my_package:1.0.0
```
This operation:

- Uploads the built ECU image and metadata,
- Registers the package in the instance’s central registry.

After pushing:

- Scientists and other users can discover and use the package in their workflows,
-  You can update the package later by pushing a new version (e.g., 1.1.0).

#### 4.4.2. Verifying published packages
You (or users) can list remote packages:

```bash
# List locally cached packages 
brane package list 
```
```bash
# Search for available packages on the remote registry
brane package search
```
This shows:

- Package names and versions,
- Whether they are local or remote,
-  Basic metadata (description, owners).

### 4.5. Managing local and remote packages
The Brane CLI distinguishes between local and remote packages.
 
- Local packages:
- Built on your machine from your source code,
- Useful for development and local testing.
- Remote packages:
- Hosted on a Brane instance’s registry,
- Available to anyone with access and permissions.

#### 4.5.1. Importing packages from Git or remote sources
The manual describes importing packages from Git repositories or remote instances:
 
- You can import a package from a GitHub repo into your local environment.
- You can pull a remote package from a Brane instance to get a local copy.
- Conceptual commands (syntax to be validated):

```bash
# Import from GitHub
brane package import https://github.com/your-org/your-package-repo.git

# Pull a remote package from an instance
brane package pull some_package:1.2.3
```
This allows you to:

- Start from existing packages,
- Inspect or modify their code locally,
- Rebuild and publish customized versions.

#### 4.5.2. Removing packages
You can clean up packages that are no longer needed.

Conceptual commands (syntax to be validated):

```bash
# Remove a local package
brane package remove my_package:1.0.0

# Unpublish a package from a remote instance
brane package unpublish my_package:1.0.0
```
These operations:

Free up local resources (images, metadata),
Remove remote packages that should no longer be used (subject to appropriate admin rights).

#### 4.5.3. Best practices for package developers
- Use semantic versioning (e.g., 1.0.0, 1.1.0) to make upgrades explicit.
- Keep container.yml in version control along with your code.
- Test locally before publishing:
- Verify inputs/outputs,
- Check performance and error handling.
- Document:
- Expected inputs and outputs,
- Dataset requirements,
- Any side-effects (e.g., external services used).


### 4.6. Summary
For software engineers, Brane provides:

- A package model (ECUs) where you package your logic as containers.
- A manifest (container.yml) describing metadata, dependencies, files, entrypoint, and actions.
- CLI workflows to:
- Build and test packages locally,
- Publish packages to a Brane instance,
- Import, pull, and remove packages from various sources.
 
These packages become the building blocks that scientists use in their workflows (see Section 6), and their correct definition and behaviour are essential for reliable, policy-aware execution in Brane.


### 4.7. Developer Quick Reference

This section summarizes the minimum you need to develop and test a Brane package.

#### 4.7.1. Minimal `container.yml` template (Python example)

```yaml
name: my_package
version: 0.1.0
kind: ecu

description: "Minimal example Brane package."
owners:
  - "Your Name <you@example.org>"

image: "python:3.11-slim"

dependencies:
  ubuntu:
    - python3
    - python3-yaml

files:
  - source: "./src"
    target: "/opt/my_package/src"

env:
  - name: PYTHONUNBUFFERED
    value: "1"

install:
  - "pip install pyyaml"

entrypoint:
  kind: "task"
  exec: "python3 /opt/my_package/src/main.py"
```

Directory structure:

```text
my-package/
  container.yml
  src/
    main.py
```
#### 4.7.2. I/O contract for package code (conceptual)
Brane packages are expected to follow a simple I/O pattern:

Input:
-  Provided as a YAML structure (e.g., via an INPUT environment variable or file).
  -  Your code parses this YAML to get function arguments.
-  Output:
  -  Write a YAML response to stdout.
  -  Optionally write a result file or directory to a known path (commonly /result) for Brane to capture.

Example main.py (conceptual):

```python
import os
import sys
import yaml

def main():
    # Read input YAML from environment variable (pattern may vary by Brane version)
    input_yaml = os.environ.get("INPUT", "{}")
    data = yaml.safe_load(input_yaml)

    # Extract arguments
    text = data.get("text", "")
    times = int(data.get("times", 1))

    # Do work
    result = text * times

    # Prepare output as YAML
    output = {"result": result}

    # Write YAML to stdout
    yaml.safe_dump(output, sys.stdout)

    # Optionally write a file result (Brane can capture /result or similar)
    os.makedirs("/result", exist_ok=True)
    with open("/result/output.txt", "w") as f:
        f.write(result)

if __name__ == "__main__":
    main()
```

Note: The exact input/output conventions (e.g., which env vars or paths to use) may depend on the current Brane version and its runtime contract. Adjust according to your deployment’s documentation.

#### 4.7.3. Typical lifecycle (commands to be validated)
1. Build locally

```bash
# In the package directory with container.yml
brane package build .
```
2. Test locally

```bash
# Test a function exposed by the package
brane package test my_package:0.1.0 my_function \
    --input '{"text": "Hello", "times": 3}'
```
Check:

-  stdout (YAML output),
-  any files under /result inside the container.

3. Publish to an instance

```bash
# Assuming instance and credentials are configured
brane package push my_package:0.1.0
```

4. List packages (local/remote)

```bash
brane package list
```
All command examples above are structural and may need updating to match the current brane CLI. Use them as patterns, not exact syntax, until we revise them with up‑to‑date notes.

#### 4.7.4. Troubleshooting checklist
Build fails:
-  Check base image and dependencies/install commands.
-  Ensure paths in files exist and are correct.
-  Package runs but crashes:
-  Log any exceptions and print them to stderr.
-  Verify that input YAML contains the keys you expect.
-  Check that your code can write to /result (permissions, paths).
-  Published package not visible:
-  Confirm you are connected to the correct instance.
-  Check registry health on the Brane instance.
-  Ensure version (version in container.yml) is unique and correctly specified.
-  These patterns cover the common path from “new package skeleton” to “published, usable component” in Brane workflows.
