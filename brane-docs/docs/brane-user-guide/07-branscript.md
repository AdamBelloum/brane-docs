# BraneScript Language Reference Guide

**Audience:** Data Scientists, Computational Analysts, and Workflow Architects  
**Purpose:** A comprehensive reference manual for writing, structuring, and executing distributed workflows using the BraneScript domain-specific language (DSL).

---

## 1. Type System & Data Structures

BraneScript is a statically typed language at runtime, ensuring that data passed between isolated container functions matches structural specifications.

### 1.1 Primitive Types
*   **`integer`**: 64-bit signed whole numbers (e.g., `42`, `-7`).
*   **`real`**: 64-bit floating-point numbers (e.g., `3.14`, `-0.001`).
*   **`string`**: UTF-8 encoded text wrapped in double quotes (e.g., `"hello"`).
*   **`boolean`**: Logical states written explicitly as `true` or `false`.

### 1.2 Composite Objects
When a package function returns a structured object rather than a primitive, fields are accessed using standard dot-notation (`.`):
```branescript
// Accessing properties of a composite object
analysis_results = data_cruncher::generate_stats(dataset);
mean_value = analysis_results.mean;

```

### 1.3 Data Assets (`Data` Type)

Datasets are treated as distinct first-class types. Variables pointing to data assets hold an underlying reference to physical files managed by the cluster registry, rather than raw in-memory strings:

```branescript
// Assigning a registered infrastructure dataset to a variable
patient_cohort = data::load("clinical-trial-2026");

```

---

## 2. Structural Syntax Rules & Imports

### 2.1 Package Registration

Before any containerized function can be invoked within a workflow script, its parent package must be explicitly declared at the top of the file using the `import` statement.

```branescript
import matrix_ops;
import visualization_tool;

```

### 2.2 Namespace Resolution

Functions within imported packages are called using the explicit namespace operator `::`:

```branescript
// Syntax: package_name::function_name(arguments)
scaled_matrix = matrix_ops::scale_dimensions(matrix_asset, 2.5);

```

---

## 3. Control Flow & Logical Operations

BraneScript supports fundamental control structures to allow conditional data processing and iterative tasks.

### 3.1 Conditional Execution (`if / else`)

Enables workflows to branch dynamically based on intermediate calculation properties:

```branescript
import threshold_checker;

score = threshold_checker::evaluate(patient_cohort);

if (score > 0.85) {
    println("High risk threshold breached. Executing deep analysis pipeline...");
    result = deep_pipeline::run(patient_cohort);
} else {
    println("Normal risk profile detected. Executing standard protocol.");
    result = standard_pipeline::run(patient_cohort);
}

```

### 3.2 Iterative Loops (`for` and `while`)

Used for processing series of parameters, epochs, or convergence states:

```branescript
// Iterating over a bounded range
for (i in 0..5) {
    println("Processing iteration index: " + text_utils::to_string(i));
    current_weights = training_module::step(model_asset, i);
}

// Looping based on evaluation criteria
while (quality_metric < 0.99) {
    model_asset = refinement_tool::optimize(model_asset);
    quality_metric = refinement_tool::evaluate(model_asset);
}

```

---

## 4. Advanced Data Flow: Lineage & Inter-Function Pipelining

One of BraneScript's primary jobs is managing dependency lineages. When the output of one function serves as the input to the next, the runtime builds a directed acyclic graph (DAG) to optimize cross-node transfers.

### 4.1 Explicit Intermediate Handshakes

You can chain computations by passing the variable holding a previous function's output straight into a subsequent function call:

```branescript
import genomic_sequencer;
import alignment_tool;
import variant_caller;

// 1. Extract raw sequencing reads (Generates a secure intermediate reference)
raw_reads = genomic_sequencer::extract_sample("sample_id_99");

// 2. Pass intermediate reference directly to the aligner
aligned_bam = alignment_tool::map_to_genome(raw_reads, "hg38_reference");

// 3. Complete pipeline by extracting variants
final_vcf = variant_caller::identify_mutations(aligned_bam);

```

---

## 5. Execution Reference & Diagnostics

### 5.1 Sandbox Verification Mode

To validate your BraneScript's syntax, loops, and functional logic locally on your workstation without committing jobs to a live production cluster, use:

```bash
brane test path/to/workflow.bs

```

This runs a clean compilation check, confirms package availability, and mocks execution.

### 5.2 Distributed Execution Mode

To submit the written script to the orchestrator for full-scale cluster execution across remote worker sites:

```bash
brane workflow run path/to/workflow.bs

```

### 5.3 Post-Execution Audit Logging

If a pipeline stops mid-execution or yields anomalous metrics, you can trace the explicit container outputs using the workflow log daemon:

```bash
# List unique historical tracking IDs for your workflows
brane workflow list

# Extract stderr/stdout streams from the remote container runtimes
brane workflow logs <WORKFLOW_TRACKING_ID>

```

```

```
