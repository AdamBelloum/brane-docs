# BraneScript Comprehensive Reference Guide

## 0. Overview

BraneScript is a **domain-specific language (DSL)** for describing workflows in the Brane framework. It is designed to:

- **Glue together package functions** (external code units) into high-level workflows.
- **Model data flow and control flow** over datasets and tasks.
- Compile to a common **Workflow Internal Representation (WIR) graph**, which the Brane orchestration layer executes locally or across distributed domains.

Brane aims to be accessible to different roles (software engineers, data scientists, policy officers) by separating concerns:

- **BraneScript**: scripting-like DSL (similar to Bash/Lua/Python), aimed at users comfortable with programming.
- **Bakery**: more natural-language-like DSL for scientists with limited programming background.

Under the hood, both DSLs **translate to the same internal representation and share semantics**; only their syntax differs.

This guide focuses on **BraneScript**.


---

## 1. Audience and Purpose

### 1.1 Audience

- Data Scientists  
- Computational Analysts  
- Workflow Architects  
- Software Engineers working with Brane workflows

### 1.2 Purpose

This guide is a **practical reference** for:

- Writing BraneScript workflows.
- Understanding BraneScript types, control flow, and data assets.
- Using Brane packages and datasets in local and distributed settings.

It synthesizes:

- Tutorial-style material (“bootcamp” chapters),
- More detailed language descriptions,
- And concrete workflow examples (e.g., the Disaster Tweets pipeline).

For exhaustive formal details, consult the **BraneScript language specification** in the *Brane: A Specification* book and the extended BraneScript documentation.


---

## 2. Brane Ecosystem and Packages

### 2.1 Brane Workflows and DSLs

The Brane framework revolves around **workflows**: high-level descriptions of algorithms or pipelines that specify:

- Which **package functions** to call,
- In what **order**,
- With which **datasets** and parameters.

Brane provides two workflow DSLs:

- **BraneScript**  
  - Syntax resembles scripting languages (Bash, Lua, Python).  
  - Convenient for engineers and technically inclined scientists.

- **Bakery**  
  - Designed to read more like natural language.  
  - Targets scientists and domain experts less familiar with programming.

Both compile to the same WIR and share semantics.

In this documentation, we use:

- **workflow** and **script** interchangeably to refer to a BraneScript file.
- **package functions** or **external functions** for functions implemented by packages rather than BraneScript itself.

### 2.2 Package Types

Brane supports multiple **package kinds**:

- **Executable Code Unit (ECU) packages**  
  - Containers containing arbitrary code.  
  - Code is run via the `branelet` wrapper inside the container.

- **OpenAPI Standard (OAS) packages**  
  - Packages that make HTTP API requests defined in OpenAPI format.  
  - Requests are performed by `branelet`.

Planned future extensions (mentioned but not yet finalized):

- Support for **Common Workflow Language (CWL)** workflows as packages.
- Publishing Brane’s DSLs (**BraneScript** and **Bakery**) as packages themselves.

### 2.3 Package Metadata (`container.yml`)

Packages are described by a `container.yml` file, which defines:

- Package name, version, and kind (ECU, OAS, …).
- Tasks/functions exposed to workflows.
- Inputs and outputs for each task.

Full documentation for `container.yml` is planned; refer to package examples and `brane package inspect` for practical insight.


---

## 3. BraneScript Files and Comments

### 3.1 File Naming

BraneScript workflows are plain text files, conventionally named:

- `something.bs` or  
- `something.bscript`.

Example:

```text
hello_world.bs
disaster_tweets.bs
```

### 3.2 Comments

Comments start with `//` and continue until end-of-line:

```text
// This is a comment
// HELLO WORLD.bs
//   by Rick Sanchez
//
// A BraneScript workflow for printing "Hello, world!" to the screen.
```

Use header comments to document what a workflow does.

### 3.3 Statement Terminator

**Every BraneScript statement must end with a semicolon** `;`.

Example:

```text
import hello_world;
let foo := 42;
println(foo);
```

Forgetting semicolons typically causes syntax errors (often reported as “end-of-file” or missing parenthesis errors).


---

## 4. Type System and Data Structures

BraneScript is a **compiled**, typed, scripting-like language:

- It provides a scripting experience.
- It compiles to a **WIR graph** with explicit data and dependency lineages.
- Types are used for **static analysis** (catching mistakes early).

### 4.1 Primitive Types and Literals

BraneScript supports common primitive literals:

- **Booleans**:  
  - Literals: $$\text{true}$$, $$\text{false}$$  
- **Integers** (64-bit signed):  
  - Example: $$42$$, $$-7$$  
- **Real numbers** (64-bit floating-point):  
  - Example: $$3.14$$, $$-0.001$$  
- **Strings** (UTF‑8, double-quoted):  
  - Example: $$"Amy"$$, $$"hello"$$  
- **Version triplets**:  
  - Example: $$1.0.0$$  
- **Null**:  
  - Example: $$\text{null}$$

Primitive type keywords used in class definitions include `string`, `bool`, etc.

### 4.2 Variables

Variables behave as in many typed languages:

- They are **typed**: one variable stores values of a single type.
- They must be **declared** before use.
- They can later be **updated**.

#### 4.2.1 Declaration

Syntax:

```text
let <ID> := <EXPR>;
```

Examples:

```text
let foo := 42;
let bar := "Hello, world!";
let flag := true;
let total := 21 + 21;
```

`<EXPR>` can be:

- Literals: $$42$$, $$"hello"$$, $$\text{true}$$
- Function calls: `hello_world()`
- Other variables: `foo`
- Operators: `foo * 2`, `a == b`, etc.

#### 4.2.2 Assignment (Update)

Syntax:

```text
<ID> := <EXPR>;
```

Example:

```text
let foo := 42;
println(foo);   // 42

foo := 84;
println(foo);   // 84

foo := foo * 2; // read foo, compute foo*2, then update
```

Variables aren’t updated until the right-hand expression is evaluated, so reading a variable within its own update is well-defined.

### 4.3 Functions (Calling and Defining)

BraneScript uses a common function call syntax:

```text
<ID>( <ARG1>, <ARG2>, ... )
```

- `<ID>` is the function name.
- Arguments are expressions, separated by commas.
- The function evaluates to its **return value** as part of the expression.

Example (calling):

```text
println("Hello, world!");  // builtin function
println(hello_world());    // external package function
```

Functions are called the same way whether they are:

- **Imported package functions** (external),
- **Builtins** (`println`, `commit_result`, `len`, …),
- **User-defined functions** (`func foo(...) { ... }`).

#### 4.3.1 Importing Package Functions

There are two common patterns:

1. **Global import** (functions become available by name):

   ```text
   import hello_world;

   println(hello_world());
   ```

   This treats `hello_world` as a function directly in the global namespace.

2. **Namespaced calls** (explicit `package::function`):

   ```text
   import matrix_ops;

   scaled_matrix := matrix_ops::scale_dimensions(matrix_asset, 2.5);
   ```

   This emphasizes the originating package and helps avoid naming conflicts.

Depending on package design and Brane version, both may be supported; adopt one style consistently in your workflows and consult current Brane documentation to see how imports are resolved in your environment.

#### 4.3.2 Function Definitions

User-defined functions:

```text
func <ID>( <ARG1>, <ARG2>, ... ) {
    <STATEMENTS>
}
```

- Arguments (`<ARG1>`, `<ARG2>`, …) act as **local variables**, initialized with the passed values.
- The function body contains statements executed when the function is called.

Examples:

```text
func print_hello_world() {
    println("Hello, world!");
}

func print_text(text) {
    println(text);
}

func print_greeting_place(greeting, place) {
    print(greeting);
    print(", ");
    print(place);
    println("!");
}
```

#### 4.3.3 Return Statement

Syntax:

```text
return <EXPR>;
```

- Immediately exits the function.
- The function call evaluates to the returned value.
- If no value is needed, `return;` (without expression) is allowed.

Example:

```text
func zero() {
    return 0;
}

func add(lhs, rhs) {
    return lhs + rhs;
}

let forty_two := add(add(2, add(zero(), 20)), 20);
println(forty_two);   // 42
```

Also:

```text
func greet(person) {
    if (person == "stinky") {
        println("That is rude, I won't print that.");
        return;
    }

    print("Hello, ");
    print(person);
    println("!");
}
```

##### Return at Workflow Level

BraneScript allows `return` in the **main workflow** (top level):

- `return;` exits the workflow early (useful in loops).
- `return <EXPR>;` returns a value from the workflow.

Example:

```text
return "A special value";
```

This can be used by the Brane CLI to capture workflow results (useful with datasets and packaged workflows).


### 4.4 Arrays

Arrays are **homogeneous ordered collections** of values.

#### 4.4.1 Array Literals

Syntax:

```text
[ <EXPR1>, <EXPR2>, ... ]
```

Examples:

```text
let value := -5;
let array := [1, value, zero()];   // [1, -5, 0]

let matrix := [ [0, 1, 2], [3, 4, 5], [6, 7, 8] ];
```

All elements in an array must have the **same type**; mixed-type arrays cause errors:

```text
// Error: mixed types
let uh_oh := [42, "forty two", 42.0];

// Correct: homogeneous
let ok := [83, 112, 97, 109];
```

#### 4.4.2 Indexing and Assignment

Arrays are **zero-indexed**:

Syntax:

```text
<ARRAY-EXPR>[ <INDEX-EXPR> ]
```

Examples:

```text
let array1 := [1, 2, 3];
println(array1[0]);         // 1

let index1 := 2;
println(array1[index1]);    // 3

println([4, 5, 6][1]);      // 5
println(generate_array()[0]);
println(array1[zero()]);    // 1 if zero() returns 0
```

Assignment via indexing:

```text
let array2 := [7, 8, 9];
array2[0] := 42;
println(array2);           // [42, 8, 9]
```

#### 4.4.3 Length

Builtin:

```text
len(<array>)
```

Example:

```text
println(len([0, 0, 0]));   // 3
```

Useful for iterating with `for` loops.


### 4.5 Classes (Composite Types)

Classes are **heterogeneous structured objects** with named fields; they can also define methods (functions associated with the class).

#### 4.5.1 Class Definition

Syntax:

```text
class <ID> {
    <FIELD-ID-1>: <FIELD-TYPE-1>;
    <FIELD-ID-2>: <FIELD-TYPE-2>;
    ...
}
```

Example:

```text
class Jedi {
    name: string;
    lightsaber_colour: string;
    is_master: bool;
}
```

Each class definition introduces a distinct **type**. Assigning instances of different class types to each other is generally not allowed.

#### 4.5.2 Instantiation

Syntax:

```text
new <ID> {
    <FIELD-ID-1> := <EXPR1>,
    <FIELD-ID-2> := <EXPR2>,
    ...
}
```

- Uses commas `,` between field assignments.
- Field order does not need to match the definition order.

Examples:

```text
let anakin := new Jedi {
    name := "Anakin Skywalker",
    lightsaber_colour := "blue",
    is_master := false,
};

let obi_wan := new Jedi {
    lightsaber_colour := "blue",
    name := "Obi-Wan Kenobi",
    is_master := true,
};
```

#### 4.5.3 Projection (Field Access)

Syntax:

```text
<CLASS-EXPR>.<FIELD-ID>
```

Examples:

```text
func print_jedi(jedi) {
    print(jedi.name);
    print(" swishes his ");
    print(jedi.lightsaber_colour);
    print(" lightsaber ");
    if (jedi.is_master) {
        println("masterfully!");
    } else {
        println("amateurishly!");
    }
}

print_jedi(anakin);
print_jedi(obi_wan);

// Assigning fields
anakin.lightsaber_colour := "green";
print_jedi(anakin);
```

BraneScript also supports methods on classes (OOP patterns), though method definition syntax is not fully detailed here; refer to extended documentation for OOP usage.


### 4.6 Data Assets (`Data` Type)

Brane treats datasets as **first-class assets**, similar to packages:

- Stored and managed by the framework.
- Governed by policy (e.g., which domain may access which dataset).

BraneScript workflows refer to datasets via a builtin `Data` class.

Example:

```text
let train := new Data{ name := "nlp_train" };
let test  := new Data{ name := "nlp_test" };
```

- `name` corresponds to the dataset identifier registered in the framework (`brane data list`).
- The `Data` object is a **reference** to the dataset, not the data itself.
- The framework attaches appropriate dataset paths to package tasks behind the scenes.

Datasets returned or promoted via `commit_result` become visible as new assets.


---

## 5. Control Flow

BraneScript supports standard control structures plus workflow-specific ones.

### 5.1 If-Statements

Syntax:

```text
if (<EXPR>) {
    <STATEMENTS>
}
```

If `<EXPR>` evaluates to `true`, the block executes; otherwise it is skipped.

Example:

```text
let some_value := 42;

if (some_value == 42) {
    println("some_value was 42!");
}
```

#### 5.1.1 If–Else

Syntax:

```text
if (<EXPR>) {
    <STATEMENTS>
} else {
    <OTHER-STATEMENTS>
}
```

Example:

```text
let some_value := 42;

if (some_value == 42) {
    println("some_value was 42!");
} else {
    println("some_value was not 42 :(");
}
```

BraneScript currently **does not have `else if` syntax**. You can emulate it with nested `if` in `else` blocks:

```text
if (some_value == 42) {
    println("some_value was 42!");
} else {
    if (some_value == 43) {
        println("some_value was 43!");
    } else {
        if (some_value == 44) {
            println("some_value was 44!");
        } else {
            println("some_value had some other value :(");
        }
    }
}
```

### 5.2 For-Loops

Syntax (C-like):

```text
for (<STATEMENT>; <EXPR>; <STATEMENT>) {
    <STATEMENTS>
}
```

- First `<STATEMENT>`: initializer, run once before the loop.
- `<EXPR>`: condition, evaluated at the start of each iteration.
- Last `<STATEMENT>`: increment, run at the end of each iteration.

Example:

```text
for (let i := 0; i < 10; i := i + 1) {
    println("Hello, world!");
}
```

This prints “Hello, world!” ten times.

> Note: In future versions, `for` syntax may be restricted or simplified; conceptually it is similar to a `while` with manual initialization and increment.

### 5.3 While-Loops

Syntax:

```text
while (<EXPR>) {
    <STATEMENTS>
}
```

Executes the block as long as `<EXPR>` evaluates to `true`.

Example equivalent to the `for` loop above:

```text
let i := 0;
while (i < 10) {
    println("Hello, world!");
    i := i + 1;
}
```

More realistic:

```text
let err := 100.0;
while (err > 1.0) {
    train_some_network();
    err := compute_error();
}
```

Infinite loop:

```text
print("The");
while (true) {
    print(" end is never the");
}
```

BraneScript **does not currently support `break`**; use:

- A Boolean flag to control loop continuation, or
- `return` (to exit the loop by exiting the function or workflow).

### 5.4 Return (Control Flow)

Covered in §4.3.3. Summary:

- `return;` exits the function or workflow.
- `return <EXPR>;` exits with a value.

Return inside loops is the primary way to perform early exit in BraneScript.

### 5.5 Parallel Statements

A distinctive BraneScript feature is the `parallel` statement, which runs **multiple branches concurrently**.

#### 5.5.1 Basic Syntax

```text
parallel [{
    <STATEMENTS>
}, {
    <MORE-STATEMENTS>
}, ...];
```

Example:

```text
parallel [{
    println("This is printed...");
}, {
    println("...while this is printed...");
}, {
    println("...at the same time this is printed!");
}];
```

Conceptually:

- Each branch runs concurrently (order is **not guaranteed**).
- The overall workflow continues **after all branches complete**; the end of the `parallel` acts as a **join point**.

#### 5.5.2 Variable Semantics

Inside `parallel`:

- Each branch gets its **own copy** of variables.
- Changes are **local to the branch** and do not escape to the outer scope.

Example:

```text
let value := 42;
parallel [{
    println(value);   // 42
}, {
    value := 84;
    println(value);   // 84
}];
println(value);       // Still 42
```

#### 5.5.3 Return from Parallel Branches

Returning in a branch:

```text
parallel [{
    println("1");
    return;
    println("2");   // never reached
}];
println("3");
```

This prints `1` and `3`:

- `return` exits the branch, not the whole workflow.
- Only when all branches finish does the workflow continue after the `parallel` statement.

#### 5.5.4 Parallel as an Expression (Collecting Results)

You can declare a variable that collects branch results:

```text
let jedis := parallel [{
    return "Obi-Wan Kenobi";
}, {
    return "Anakin Skywalker";
}, {
    return "Master Yoda";
}];
println(jedis);
```

This yields an **array** of returned values. Since branch completion order is unspecified, the order in the array is **undefined** (first-come-first-served).

#### 5.5.5 Merge Strategies

Parallel statements support **merge strategies** that define how branch results are combined:

Syntax:

```text
let res := parallel [<STRATEGY>] [{
    <STATEMENTS>
}, {
    <MORE-STATEMENTS>
}];
```

Example from the tutorial:

```text
let res := parallel [all] [{
    return 42;
}, {
    return 42;
}];
println(res);   // prints 84
```

In this example, the strategy labeled `[all]` behaves like a **sum strategy**, adding the numeric results. The documentation text calls it a “sum-strategy” but uses the identifier `all`; consult current BraneScript reference for the exact strategy names and semantics.

Other strategies are available (e.g., collecting into arrays); see official docs for a complete list.


---

## 6. Builtin Functions

Common builtins include:

- `print(<value>)`  
  - Prints a value to stdout **without** newline.
- `println(<value>)`  
  - Prints a value to stdout **with** newline.
- `len(<array>)`  
  - Returns length of an array (integer).
- `commit_result(<string>, <result>)`  
  - Promotes an intermediate result to a **dataset asset** with the given identifier.

`println` and `print` accept any value; arguments are serialized to strings before printing.

`commit_result` is central to managing **dataset lifecycle**: intermediate results become reusable, policy-governed datasets.


---

## 7. Execution: Local, Remote, and REPL

Brane workflows can be executed:

- **Locally**: tasks run on your machine, using locally installed packages and datasets.
- **Remotely**: tasks run on domains in a Brane instance (e.g., clusters at UvA, SURF).

### 7.1 Local Execution

#### 7.1.1 Prerequisites

- Brane executable installed and on `PATH`.
- Docker installed and running.
- Required packages installed locally.

Check packages:

```bash
brane package list
```

Example package import (from GitHub):

```bash
brane package import braneframework/brane-std hello_world/container.yml
```

Run a workflow:

```bash
brane workflow run hello_world.bs
# or
brane workflow run <PATH_TO_WORKFLOW>
```

Output includes:

- Printouts from `println`.
- Status messages and any errors.
- Possibly a path to downloaded result datasets (if returned or committed).

> Note: Even simple workflows may take a few seconds the first time, due to container startup.

#### 7.1.2 Sandbox Verification

To validate syntax and logic without full cluster execution, you can use:

```bash
brane test path/to/workflow.bs
```

This checks the workflow in a sandbox mode.

### 7.2 Remote Execution

#### 7.2.1 Instance Registration

First, register a remote instance:

```bash
brane instance add brane01.lab.uvalight.net --name tutorial --use
brane instance list --show-status
```

- `--name tutorial` assigns a local nickname.
- `--use` selects the instance for subsequent remote commands.

Ensure required packages are available on the instance (`brane package search`, `brane package push`)—though in tutorials this may be preconfigured.

#### 7.2.2 Certificates

To authenticate with remote domains, add certificates per domain:

```bash
# University of Amsterdam
brane certs add ./ca.pem ./client-id.pem --domain uva

# SURF
brane certs add ./ca.pem ./client-id.pem --domain surf

brane certs list
```

Certificates are typically provided by domain administrators; tutorials may supply shared test keys.

#### 7.2.3 Choosing Execution Location: `on` Blocks

If multiple domains have access to the same datasets, the planner may require explicit location choice:

```text
import compute;
import visualization;

on "uva" {
    // your workflow code
}
```

- `"uva"`: UvA OpenLab cluster.
- `"surf"`: SURF ResearchCloud environment.

`on "<domain>" { ... }` ensures tasks inside the block run on the chosen domain. Import statements can remain outside.

> The `on` block affects tasks (package function calls), not builtins like `commit_result`—results may be replicated or downloaded from different locations depending on policy and deployment.

#### 7.2.4 Remote Workflow Execution

Run the workflow remotely:

```bash
brane workflow run --remote hello_world.bs
# or
brane workflow run <PATH_TO_WORKFLOW> --remote
```

Results should be similar to local execution, but tasks run on remote domains; datasets returned may be automatically downloaded.

Use `--debug` to inspect remote data transfers and where results are fetched from.

### 7.3 Read–Eval–Print Loop (REPL)

BraneScript REPL:

```bash
brane workflow repl
```

Then in the REPL:

```text
import hello_world;
println(hello_world());
```

This allows interactive experimentation.

Remote REPL:

```bash
brane workflow repl --remote
```

> Note: Due to policies and authorization, behavior may differ between REPL and file-based workflows. Some policies rely on knowing the full workflow up front, which is harder in REPL mode.

---

## 8. Example: Simple “Hello, world!” Workflow

A minimal workflow:

```text
// HELLO WORLD.bs
//   by Rick Sanchez
//
// A BraneScript workflow for printing "Hello, world!" to the screen.

// Import package that defines 'hello_world()'
import hello_world;

// Print the result of 'hello_world()' using 'println()'
println(hello_world());
```

Run locally:

```bash
brane workflow run hello_world.bs
```

Run remotely (after instance setup):

```bash
brane workflow run --remote hello_world.bs
```


---

## 9. Example: Disaster Tweets Classification Workflow

This example demonstrates a more complex, end-to-end BraneScript workflow using existing packages for NLP on the **Disaster Tweets** dataset.

### 9.1 Objective

Implement a workflow that:

1. Cleans and preprocesses training and test datasets.
2. Trains a Naive Bayes classifier to predict whether a tweet describes a disaster (`1`) or not (`0`).
3. Generates visualizations of the model and dataset in an HTML file.

### 9.2 Prerequisites

- Brane executable installed.
- Docker installed.
- Required packages and datasets prepared.

### 9.3 Building Packages

Use `brane import` to build two packages from a GitHub repository:

```bash
brane import epi-project/brane-disaster-tweets-example -c packages/compute/container.yml
brane import epi-project/brane-disaster-tweets-example -c packages/visualization/container.yml

brane package list
```

Packages:

- `compute`: preprocessing and model training tasks.
- `visualization`: plotting and HTML report generation.

### 9.4 Building Data Assets

Each dataset (train and test) comes with a `data.yml` describing metadata and file paths.

For each dataset (run in its directory):

```bash
brane data build ./data.yml
brane data list
```

You should see identifiers:

- `nlp_train`: training set.
- `nlp_test`: test set.

By default, `brane data build` links to the original file; use `--no-links` if you want Brane to copy the dataset and you intend to delete the original file.

### 9.5 Workflow: Compute + Visualization

Create `workflow.bs`:

```text
import compute;
import visualization;

// Refer to datasets by name
let train := new Data{ name := "nlp_train" };
let test  := new Data{ name := "nlp_test" };

// Clean datasets
let train_clean := clean(train);
let test_clean  := clean(test);

// Tokenize
let train_final := tokenize(train_clean);
let test_final  := tokenize(test_clean);

// Remove stopwords
train_final := remove_stopwords(train_final);
test_final  := remove_stopwords(test_final);

// Vectorize train and test together; returns combined vectors
let vectors := create_vectors(train_final, test_final);

// Train model; returns dataset representing the model
let model := train_model(train, vectors);

// Promote model to public dataset asset
commit_result("nlp_model", model);

// Create submission: predictions for test set
let submission := create_submission(test, vectors, model);

// Generate visualizations bundled in an HTML file
let plot := visualization_action(
    train,
    test,
    submission
);

// Commit plot dataset and return it so client downloads results
return commit_result("nlp_plot", plot);
```

Notes:

- `train`, `test` are `Data` references, not raw CSV.
- Intermediate datasets (`train_clean`, `train_final`, `vectors`, `model`) may be **temporary**, unless promoted via `commit_result`.
- By returning `commit_result("nlp_plot", plot);`, the workflow instructs the client to download the resulting HTML dataset.

### 9.6 Local Execution and Debugging

Run locally:

```bash
brane run workflow.bs
# or
brane workflow run workflow.bs
```

Use `println` to monitor progress:

```text
println("Starting preprocessing...");
println(train);

println("Vectorization complete");
```

After successful run:

- The client will show where `nlp_plot` was stored.
- Open the `index.html` in the downloaded dataset directory (e.g., with `firefox "<PATH>/index.html"`).
- View plots summarizing the model and dataset.

### 9.7 Remote Execution with `on` Blocks

To run the workflow on a remote instance with multiple domains:

1. Wrap main workflow in an `on` block:

   ```text
   import compute;
   import visualization;

   on "uva" {
       // Refer to datasets
       let train := new Data{ name := "nlp_train" };
       let test  := new Data{ name := "nlp_test" };

       // Preprocessing, vectorization, model training,
       // submission creation, visualization, and return
       // (copy the body from §9.5 here)
   }
   ```

2. Ensure instance and certificates are configured (see §7.2).

3. Run:

   ```bash
   brane run workflow.bs --remote
   # or
   brane workflow run workflow.bs --remote
   ```

The tasks (compute + visualization) run on `"uva"`; the final plot dataset `nlp_plot` is downloaded to your local machine.

> Depending on replication and policy, the dataset might reside on multiple domains; the client may download it from different locations across runs.

---

## 10. Putting It All Together

BraneScript combines:

- **Types and data structures** (primitives, arrays, classes, `Data` assets),
- **Function calls and definitions** (package functions, builtins, user-defined functions),
- **Control flow** (`if`, `for`, `while`, `return`, `parallel`, `on`),
- **Package and dataset management** (`import`, `commit_result`, `brane package/data` commands),
- **Execution modes** (local, remote, REPL),

into a single, coherent language for specifying workflows.

Use this guide as:

- A **reference** while writing and reviewing workflows.
- A **bridge** between tutorial examples (e.g., Hello World, Disaster Tweets) and more advanced applications.
- A base for exploring advanced chapters (e.g., OOP with classes, merge strategies in parallel, policy-aware data movement) in the full BraneScript documentation and specification.

```
