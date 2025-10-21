# BraneScript User Guide

_Last updated: October 2025_  
**Audience:** Scientists, data analysts, or anyone composing workflows  
**Purpose:** Learn how to use BraneScript to define, run, and explore workflows in Brane.  

---

##  1. Introduction to BraneScript

**BraneScript** is a simple, domain-specific language (DSL) for defining workflows in Brane.  
It allows you to **compose, connect, and orchestrate** functions from different Brane packages into complete data processing pipelines.

###  Key Features

- Intuitive syntax similar to modern scripting languages  
- Portable across Brane deployments (local or distributed)  
- Supports **variables**, **function calls**, **control flow**, and **data passing**  
- Integrates directly with Brane packages and datasets  

BraneScript files use the `.bs` extension and are executed with the `brane` CLI.

---

## 2. BraneScript Basics

A minimal BraneScript program looks like this:

```branescript
// hello.bs
import hello_world;

println(hello_world::say_hello());
```

### Syntax Essentials

| Element           | Description                | Example                            |
| ----------------- | -------------------------- | ---------------------------------- |
| **Import**        | Load a package or module   | `import hello_world;`              |
| **Function call** | Execute a package function | `hello_world::say_hello();`        |
| **Assignment**    | Store output of a function | `msg = hello_world::greet("Ada");` |
| **Print**         | Display values             | `println(msg);`                    |
| **Comments**      | Start with `//`            | `// This is a comment`             |

---

### Example with Data Flow

```branescript
import math_ops;
import text_utils;

a = 5;
b = 3;
sum = math_ops::add(a, b);
msg = text_utils::to_string(sum);

println("The result is: " + msg);
```

---

## 3. Importing Packages and Calling Functions

Each package defines a set of **functions** (or operations).
You import the package and call its functions using the `package::function()` syntax.

### Example

```branescript
import hello_world;

println(hello_world::say_hello());
```

If a function takes parameters:

```branescript
import hello_world;

message = hello_world::greet("Ada");
println(message);
```

If it produces structured outputs:

```branescript
result = data_analysis::compute_average(dataset);
println(result.mean);
```

> Tip: Use `brane packages list` to view all available packages.

---

## 4. Running Scripts Locally or Remotely

You can execute BraneScript workflows using the `brane` CLI.

### Run Locally

```bash
brane workflow run hello.bs
```

Brane compiles your script, resolves the required packages, and executes all steps on your local instance.

### Run on a Distributed Setup

If your Brane installation includes remote worker nodes, simply run the same command Ñ
Brane automatically orchestrates execution across the available infrastructure.

```bash
brane workflow run hello.bs
```

To view available instances:

```bash
brane instance list
```

To select a specific environment:

```bash
brane workflow run hello.bs --instance <instance_name>
```

---

## 5. Using the Interactive REPL

Brane provides a **REPL (ReadÐEvalÐPrint Loop)** to experiment with BraneScript interactively.

Start the REPL:

```bash
brane workflow repl
```

You will see:

```
Welcome to the BraneScript REPL!
Type :help for assistance.
```

### Example Session

```
>>> import hello_world;
>>> println(hello_world::say_hello());
Hello, world!
```

Exit with:

```
:quit
```

>The REPL is great for quick exploration Ñ testing functions, learning syntax, or debugging.

---

## 6. Example: ÒHelloWorldÓ Workflow

HereÕs a complete BraneScript example you can try right away.

Create a file `HelloWorld.bs`:

```branescript
// HelloWorld.bs
// Demonstrates calling a package function and printing output

import hello_world;

println(hello_world::say_hello());
```

Run it:

```bash
brane workflow run HelloWorld.bs
```

Expected output:

```
Hello, world!
```

---

### Example with Variables and Composition

```branescript
import hello_world;
import text_utils;

// Call a package function
msg = hello_world::greet("Ada");

// Manipulate the string
upper_msg = text_utils::to_upper(msg);

// Display the result
println(upper_msg);
```

---

## 7. Understanding Execution Flow

When you run a BraneScript:

1. The script is parsed and validated.
2. Brane identifies the required packages and their locations.
3. Tasks are distributed to nodes (local or remote).
4. Each function runs inside a secure container.
5. Outputs are collected and made available for subsequent steps.

You can inspect logs or workflow results using:

```bash
brane workflow list
brane workflow logs <workflow_id>
```

---

## Summary

| Task           | Command                        | Purpose                              |
| -------------- | ------------------------------ | ------------------------------------ |
| Run a script   | `brane workflow run <file>.bs` | Execute workflow locally or remotely |
| Use REPL       | `brane workflow repl`          | Try commands interactively           |
| Import package | `import <package>;`            | Access package functions             |
| Call function  | `<package>::<function>(args)`  | Execute package code                 |
| Print output   | `println()`                    | Display workflow results             |

---

**BraneScript lets you automate complex data workflows in a simple, reproducible way.**
Next, learn how to [Administer and Manage Brane Nodes ?](admin-guide.md)

[? Back to Table of Contents](brane-docs-index.md)
