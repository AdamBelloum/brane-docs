# 5. Workflow Internal Representation (WIR)

Brane uses a **workflow internal representation (WIR)** as the common language for workflows.

Frontend languages (e.g., BraneScript) compile into WIR. The runtime (VM) executes WIR graphs. This separation allows:

- Multiple frontends to coexist,
- A single, well-defined execution model,
- Stable internal contracts for core developers.

## 5.1. Design rationale

WIR is designed to:

- Represent workflows as **graphs** with explicit data and control dependencies.
- Provide a **federated view** of workflows that span multiple domains.
- Support rich control flow (branches, loops, parallelism) without hard-wiring frontend syntax into the runtime.

Key ideas:

- **Graph-based representation**  
  Nodes represent work or control constructs; edges represent sequence, branching, parallelism, and joins.

- **Unified internal format**  
  All frontends compile into WIR; the runtime only understands WIR, not individual frontend syntax.

- **Description vs execution**  
  WIR describes what to run and how tasks depend on data and other tasks. The VM and planner decide where and when to run tasks.

## 5.2. Graph representation

A WIR program has:

- One **top-level graph** (main),
- One graph per **function**, stored in a function graph table.

### 5.2.1. Main and function graphs

- **Main graph**  
  Top-level workflow:
  - Calls functions,
  - Encodes control flow (branches, loops, parallel blocks),
  - Represents data dependencies.

- **Function graphs**  
  Each function has its own graph:
  - Local control flow,
  - Parameter handling,
  - Local variables and results.

This supports reuse and multiple calls per function.

### 5.2.2. Edge kinds and control flow

Graphs are composed of **edges** that define control flow. Key edge kinds include:

- **Linear edges** – sequential flow.
- **Node edges** – transitions to specific nodes that perform work or evaluate expressions.
- **Stop edges** – terminate a graph or path.
- **Branch edges** – conditional logic (if/else), based on stack values.
- **Parallel edges** – start parallel execution.
- **Join edges** – synchronize parallel branches.
- **Loop edges** – represent looping constructs.
- **Call edges** – invoke function graphs; push frames and transfer arguments.
- **Return edges** – return from functions; pop frames and restore caller context.

Edges can carry **edge instructions** that manipulate runtime state.

## 5.3. Symbol tables, results, and typing

### 5.3.1. Symbol table

The **symbol table** (`SymTable`) holds definitions for:

- Functions,
- Tasks,
- Classes,
- Variables.

For each symbol, it stores:

- Name and kind,
- Type,
- Metadata needed by runtime and planner.

Graphs refer to symbols by identifiers.

### 5.3.2. Results map

The **results map** links intermediate results to their locations.

For each result:

- What produced it (function or task),
- Where it is stored (domain),
- How it can be accessed (access and preprocess kinds).

This is essential for planning data movement and resolving dependencies.

### 5.3.3. Typing model

WIR is strongly typed:

- Values have types (integers, reals, strings, booleans, arrays, function handles, etc.).
- Variables are declared with types in the symbol table.
- Once a variable has a type, it cannot change type.

Edge instructions operate on typed values and include arithmetic, comparisons, array operations, and casts.

## 5.4. Example WIR for a simple workflow (conceptual)

A typical workflow:

1. Generates a dataset.
2. Transforms it.
3. Prints or exports results.

Conceptually:

- **Main graph**:
  - Calls generate, transform, and output functions.
  - Encodes control flow and dependencies.

- **Function graphs**:
  - Bind arguments to local variables.
  - Perform computations via edge instructions.
  - Call external packages as needed.
  - Return results to callers.

During execution:

- **Call edge**:
  - VM pushes a new frame,
  - Moves arguments from stack/variables to callee parameters,
  - Transfers control to the function graph.

- Inside the function:
  - Instructions manipulate stack and variables,
  - May trigger external calls and record results.

- **Return edge**:
  - Places return values where the caller expects,
  - Pops the frame,
  - Resumes caller control.

The full specification’s examples trace program counter movement, stack/frame updates, plugin interactions, and use of the results map.

## 5.5. Summary

WIR provides a stable, graph-based internal language with:

- Explicit control-flow edges,
- Strong typing and symbol tables,
- A results map connecting intermediate data to locations and access mechanisms,
- Clear separation between frontend syntax and runtime execution.

Frontends must compile into valid WIR; the VM and planner must execute WIR according to these semantics.