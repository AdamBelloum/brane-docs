# 7. BraneScript Frontend (Appendix for language developers)

BraneScript is a **workflow description language** for Brane.

- Designed for scientists and advanced users to describe data-centric workflows.
- Not the only possible frontend; other languages can target WIR.
- From the core system’s perspective, BraneScript is a **compiler frontend** that produces WIR graphs, symbol tables, and metadata.

This section is aimed at language and compiler developers.

## 7.1. Role of BraneScript relative to WIR

BraneScript sits above WIR:

- Users write workflows in BraneScript.
- The BraneScript compiler:
  - Parses source,
  - Performs semantic checks,
  - Produces WIR:
    - Main graph,
    - Function graphs,
    - Symbol table entries,
    - Results metadata (where applicable).

The runtime (VM) does not know BraneScript syntax; it only executes WIR.

## 7.2. Language overview

BraneScript offers an imperative style for expressing workflows.

Key features:

- **Declarations**
  - Functions with named parameters and return types.
  - Typed variables.

- **Expressions**
  - Literals (integers, reals, strings, booleans).
  - Arithmetic and boolean expressions.
  - Arrays and indexing.

- **Control flow**
  - Conditionals (if/else).
  - Loops (for/while-style).
  - Parallel constructs where supported.

- **Package and dataset interaction**
  - Calls to package functions (ECUs) defined in `container.yml`.
  - References to datasets from `data.yml`.

The language is strongly typed; the compiler enforces type rules before emitting WIR.

## 7.3. Core constructs (conceptual)

### 7.3.1. Functions

Functions:

- Have names, parameters, and optional return types.
- Can call other functions and package functions.

On compilation:

- Each function becomes a WIR function graph.
- Parameters and locals become symbol table entries and VM variables.

### 7.3.2. Variables and types

Variables:

- Declared with name and type.
- Use the same basic type system as WIR (primitive types, arrays, etc.).

Compiler responsibilities:

- Assign symbol IDs,
- Emit WIR instructions for initialization, `LoadVar`, and `StoreVar`.

### 7.3.3. Control flow

Control-flow constructs map to WIR edge patterns:

- **Conditionals**  
  - Compile into branch edges and join points.
  - Conditions evaluated via stack operations; edge kind chooses branch.

- **Loops**  
  - Compile into subgraphs with loop edges:
    - Initialization,
    - Condition check,
    - Body,
    - Update and back-edge.

- **Parallelism**  
  - Compile into parallel edges spawning multiple paths,
  - Join edges to synchronize.

### 7.3.4. Package calls

BraneScript calls package functions:

- Referencing package name/version and function/action.
- Providing typed arguments.

Compilation:

- Emit WIR nodes and edges that:
  - Push arguments onto the stack,
  - Trigger external ECU calls via VM and workers,
  - Record outputs in the results map.

### 7.3.5. Dataset references

Datasets from `data.yml`:

- Referenced by identifiers in BraneScript.
- Compiler maps identifiers to dataset names and metadata,
- Emits WIR structures that:
  - Create/update results map entries,
  - Use access and preprocess kinds where needed.

## 7.4. Compilation to WIR

High-level compilation stages:

1. **Parsing**
   - Parse source into an AST.

2. **Semantic analysis**
   - Name resolution and scoping.
   - Type checking.
   - Validation of function signatures and package calls.

3. **WIR generation**
   - Generate main and function graphs.
   - Allocate symbol IDs for functions and variables.
   - Translate:
     - Expressions to stack operations,
     - Variable usage to `LoadVar`/`StoreVar`,
     - Control flow to edges (branch, loop, parallel, join, stop).

4. **Linking with packages and datasets**
   - Resolve package references using `container.yml`.
   - Resolve dataset references using `data.yml`.
   - Populate results map and dataset bindings.

5. **Emission**
   - Produce a WIR program:
     - Graphs,
     - Symbol table,
     - Results map,
     - Metadata for planner and VM.

The compiler must ensure that WIR is well-typed and structurally valid.

## 7.5. Frontend evolution and alternatives

Because WIR and VM form the core contract:

- BraneScript is a **reference frontend**, not the only one.
- New DSLs or embedded languages can be implemented if they target WIR.

All frontends must:

- Compile into valid WIR,
- Respect typing and execution constraints,
- Integrate with `container.yml` and `data.yml`.

BraneScript can evolve with new features, but its compiler must preserve compatibility and the WIR contract.

## 7.6. Summary

BraneScript is a user-facing workflow language that:

- Provides typed functions, variables, control flow, and package/dataset integration,
- Compiles into WIR graphs and symbol tables,
- Relies on the VM for execution.

For core developers, BraneScript is mainly a **compiler concern**; the key is the mapping from language constructs to WIR and VM semantics.