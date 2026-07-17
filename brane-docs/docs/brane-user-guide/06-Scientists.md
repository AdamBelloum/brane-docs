## 6. Scientists: Authoring Workflows

This section is for **scientists and analysts** who want to:

- Use existing packages,
- Work with datasets,
- Write workflows in BraneScript or Bakery,
- Run workflows on a Brane instance.

It assumes:

- A Brane instance is already running (Section 3),
- Your administrator has given you **CLI access** and **client certificates** (Section 7).

> **Important:** CLI commands shown here are examples based on the current manual and may use deprecated options or subcommands. Treat them as **patterns**; we will validate exact syntax against the current CLI later.

---

### 6.1. BraneScript overview

**BraneScript** is a scripting language for describing workflows:

- It provides functions, variables, control flow (if, loops),
- It integrates with Brane packages and datasets,
- It compiles down to Brane’s internal workflow representation (WIR).

Typical BraneScript elements:

- `import` statements for packages,
- Variable declarations and assignments,
- Calls to package functions,
- Control flow (conditionals, loops),
- Return/commit of results. [1]

#### 6.1.1. Conceptual example

A very simple, conceptual BraneScript example might look like:

```branescript
import "text" version "1.0.0";

fn main() {
    let message: string = "Hello, Brane!";
    let result: string = text.uppercase(message);
    println(result);
}
```
Where:

import "text" imports a package named text,
text.uppercase calls a function defined by that package,
println outputs the result (actual function names may differ in your version).

The exact syntax and standard library functions depend on the current BraneScript version and tooling.

### 6.2. Bakery overview
Bakery provides a more declarative way to describe workflows (typically YAML-based).

With Bakery, you:

- Define steps that reference packages and functions,
- Describe how data flows between steps,
- Optionally annotate steps with metadata (e.g., hints for execution).
- Conceptually, a Bakery workflow might look like:

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
Where:

- steps define sequential or dependency-based actions,
- Each step references a package:version and a function,
- Outputs from one step can be used as inputs to later steps.

Exact Bakery syntax and features depend on the version used in your deployment.

### 6.3. Using packages in workflows
As a scientist, you usually re-use existing packages developed by software engineers (Section 4).


#### 6.3.1. Discovering available packages
Use the CLI to list packages on the active Brane instance:

```bash
# Syntax to be validated later
brane package list
```
You should see:

- Package names and versions,
- Descriptions and owners.
- If your CLI distinguishes local vs remote packages, you may be able to filter or view where each package lives (local cache vs instance registry).


#### 6.3.2. Understanding package functions and types
Each package exposes one or more functions:

Input parameters with types (e.g., string, integer, dataset reference),
Output values and/or result datasets.

You may be able to inspect package details via CLI (conceptually):

```bash
# Syntax to be validated later
brane package info my_package:1.0.0
```
This should show:

- Functions in the package,
- Parameter names and types,
- Example usage or documentation (if provided by the package author).
- You then call these functions from BraneScript or Bakery:

In BraneScript:
    - my_package.some_function(arg1, arg2)
In Bakery:
    - Steps referencing package: my_package:1.0.0 and function: some_function.

### 6.4. Working with datasets and intermediate results
Brane distinguishes between:

- Datasets – persistent data sources, described via data.yml,
- Intermediate results – data produced by workflow steps, which can also be tracked as datasets.

#### 6.4.1. Listing datasets
Use the CLI to discover datasets:

```bash
# Syntax to be validated later
brane data list
```
Information typically includes:

- Dataset names,
- Domain or location,
- Descriptions and owners.

#### 6.4.2. Using datasets in workflows
In BraneScript, you may refer to datasets by name or by handles provided by the Brane environment. Conceptually:

```branescript
import "analysis" version "1.0.0";

fn main() {
    // Assume dataset "patients" exists
    let ds = dataset("patients");   // Conceptual API
    let summary = analysis.summarize(ds);

    // Commit result dataset
    commit(summary, name = "patients_summary");
}
```
In Bakery, you might reference datasets directly in step inputs:

```yaml
steps:
  - id: summarize
    package: analysis:1.0.0
    function: summarize
    input:
      dataset: "patients"
    output:
      as_dataset: "patients_summary"
```
The exact APIs for dataset references (e.g., functions like dataset(), commit(), or YAML keys like as_dataset) depend on your BraneScript/Bakery version and environment configuration.

####  6.4.3. Intermediate results
Intermediate results are data that:

- Are produced by workflow steps,
- May be used by downstream steps,
- May or may not be committed as named datasets at the end.
 
Brane tracks intermediate results and their locations in its internal results map, but as a scientist you typically:

See them as variables in BraneScript,
Or as step outputs you can reference in later steps in Bakery.

### 6.5. Running workflows
You run workflows through the Brane CLI. The manual describes different modes (e.g., local vs remote execution).

#### 6.5.1. Running a BraneScript workflow
Assume you have workflow.bs containing your BraneScript workflow.

Conceptually:

```bash
# Syntax to be validated later
brane workflow run workflow.bs
```
Behaviour:

- The CLI compiles BraneScript into WIR,
- Sends the WIR to the Brane instance,
- The instance’s driver and planner:
- Validate the workflow,
- Plan tasks across domains,
- Execute them via delegates and workers.
 
You may also have subcommands for:

- Dry-run / validation only,
- Targeting specific domains or backends (if supported),
- Passing parameters at runtime.


#### 6.5.2. Running a Bakery workflow
For a Bakery YAML workflow file, say workflow.yaml:

```bash
# Syntax to be validated later
brane workflow run workflow.yaml
```
The process is similar:

- Bakery is compiled into WIR,
- Submitted and executed on the Brane instance.
 
#### 6.5.3. Monitoring and inspecting results
During and after execution, you may:

- View workflow status and logs:
```bash
# Syntax to be validated later
brane workflow status <workflow-id>
brane workflow logs <workflow-id>
List datasets produced:
```
```bash
brane data list
```
Download or inspect specific datasets or result files, depending on the CLI features and instance configuration.
 
### 6.6. Example workflow (conceptual)
Below is a conceptual example combining packages and datasets:

You have:
- A dataset patients,
- A package preprocess:1.0.0 with a clean function,
- A package ml:1.0.0 with a train function.
- BraneScript example:

```branescript
import "preprocess" version "1.0.0";
import "ml" version "1.0.0";

fn main() {
    let raw = dataset("patients");
    let clean = preprocess.clean(raw);
    let model = ml.train(clean);

    commit(model, name = "patients_model");
}
```

Bakery example:
```yaml
name: patients-model-workflow
steps:
  - id: clean
    package: preprocess:1.0.0
    function: clean
    input:
      dataset: "patients"
    output:
      as_dataset: "patients_clean"

  - id: train
    package: ml:1.0.0
    function: train
    input:
      dataset: "patients_clean"
    output:
      as_dataset: "patients_model"
```
Running this workflow will:

- Use the preprocess.clean function to create a cleaned dataset,
- Use ml.train to train a model,
- Commit the final model as a new dataset named patients_model.
 
### 6.7. Troubleshooting for scientists
Common issues and checks:

- Package not found:
- Use brane package list to confirm the package is available on the instance.
- Check the package name and version in your workflow.
- Dataset not found or access denied:
- Use brane data list to see available datasets.
- Confirm that your policies (Section 5) allow access to the dataset from your domain.
- Workflow fails with errors:
    - Check brane workflow logs <workflow-id> for stack traces or error messages.
    - Verify that inputs match expected types for functions.
    - Coordinate with package developers if the error originates inside a package.
    - Unexpected results:
    - Inspect intermediate datasets or step outputs where possible.
    - Simplify the workflow and test individual steps/packages.
    - For policy-related failures (e.g., denied data movement or access), contact the data policy expert / data steward responsible for the domain (Section 5).
