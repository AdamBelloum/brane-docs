```markdown
# BraneScript Comprehensive Reference Guide

## 2. Audience

Data Scientists, Computational Analysts, and Workflow Architects.

## 3. Purpose

A comprehensive reference manual for writing, structuring, and executing distributed workflows using the BraneScript domain-specific language (DSL), synthesizing official documentation specifications with practical user patterns.

---

## 1. Type System & Data Structures

BraneScript is a compiled scripting-like language designed to emulate a scripting experience while unfolding to a Workflow Internal Representation (WIR) graph.

### 1.1 Primitive Types & Literals

Literals are the simplest expressions and include:

- **Booleans**: $$\text{true}$$, $$\text{false}$$
- **Integers**: $$42$$
- **Real numbers**: $$84.0$$
- **Strings**: $$"Amy"$$
- **Version triplets**: $$1.0.0$$
- **Null**: $$\text{null}$$

Primitive types:

- **integer**: 64-bit signed whole numbers (e.g., $$42$$, $$-7$$).
- **real**: 64-bit floating-point numbers (e.g., $$3.14$$, $$-0.001$$).
- **string**: UTF-8 encoded text wrapped in double quotes (e.g., $$"hello"$$).
- **boolean**: Logical states written explicitly as $$\text{true}$$ or $$\text{false}$$.

### 1.2 Composite Objects and Arrays

- **Arrays**: Ad-hoc containers for values of a shared type, constructed using syntax like $$[1, 2, 3, 4]$$ and zero-indexed access (e.g., $$\text{foo}[0]$$).
- **Classes**: Statically sized, heterogeneous containers of multiple values defined with explicit name/type pairs. Classes can contain methods taking $$\text{self}$$ as their first parameter to support object-oriented patterns.
- **Object Property Access**: When a package function returns a structured object rather than a primitive, fields are accessed using standard dot-notation ($$.$$).

**Code snippet:**

```text
analysis_results = data_cruncher::generate_stats(dataset);
mean_value = analysis_results.mean;
```

### 1.3 Data Assets (Data Type)

Datasets are treated as distinct first-class types. Variables pointing to data assets hold an underlying reference to physical files managed by the cluster registry rather than raw in-memory strings.

**Code snippet:**

```text
patient_cohort = data::load("clinical-trial-2026");
```

---

## 2. Structural Syntax Rules & Imports

### 2.1 Package Registration

Before any containerized function can be invoked within a workflow script, its parent package must be explicitly declared at the top of the file using the $$\text{import}$$ statement, supporting optional explicit versioning.

**Code snippet:**

```text
// Import the latest available version of a package
import hello_world;

// Or import a specific version explicitly
import hello_world[1.0.0];
```

### 2.2 Namespace Resolution

Functions within imported packages are called using the explicit namespace operator $$::$$.

**Code snippet:**

```text
scaled_matrix = matrix_ops::scale_dimensions(matrix_asset, 2.5);
```

---

## 3. Control Flow & Logical Operations

BraneScript supports standard statement-driven control structures for conditional data processing and iterative tasks.

### 3.1 Conditional Execution (if / else)

Conditional execution allows divergence in control flow based on a boolean expression.

**Code snippet:**

```text
let foo := true;
let bar := null;

if (foo) {
    bar := 42;
} else {
    bar := 84;
}
```

### 3.2 Iterative Loops (for and while)

- **While-loops**: Repeat a block of statements as long as a boolean expression evaluates to true.
- **For-loops**: Syntax sugar patterned with a C-like initialization, condition, and increment expression.

**Code snippet:**

```text
// While loop example
let counter := 0;
while (counter < 10) {
    println("Hello, world!");
    counter := counter + 1;
}

// For loop example
for (let i := 0; i < 10; i := i + 1) {
    println("Hello, world!");
}
```

---

## 4. Advanced Data Flow & Lineage

One of BraneScript's primary roles is managing dependency lineages to build directed acyclic graphs (DAGs) that optimize cross-node data transfers.

### 4.1 Inter-Function Pipelining

You can chain computations by passing the variable holding a previous function's output straight into a subsequent function call.

**Code snippet:**

```text
import genomic_sequencer;
import alignment_tool;
import variant_caller;

raw_reads = genomic_sequencer::extract_sample("sample_id_99");
aligned_bam = alignment_tool::map_to_genome(raw_reads, "hg38_reference");
final_vcf = variant_caller::identify_mutations(aligned_bam);
```

### 4.2 Parallel Execution

BraneScript features explicit parallel statements to run statements concurrently, with optional merge strategies to aggregate returned values.

**Code snippet:**

```text
// Execute statements in parallel
parallel [{
    println("Hello, world! 1");
}, {
    println("Hello, world! 2");
}];

// Parallel statement returning values with a merge strategy
let result := parallel [all] [{
    return 42;
}, {
    return 84;
}, {
    return 126;
}];
```

### 4.3 Attributes and Annotations

Attributes provide additional metadata to the compiler for execution placement or policy tagging, using $$\#\[\dots\]$$ for specific statements or $$\#\!\[\dots\]$$ for parent blocks.

**Code snippet:**

```text
// Explicitly state domains/locations where a call must execute
#[on("site_1")]
{
    println("Running on site_1!");
    println(hello_world());
}

// Attach metadata tags (e.g., GDPR-like purpose tags)
#[tag("amy.research_only")]
let test1 := hello_world();
```

---

## 5. Execution Reference & Diagnostics

### 5.1 Sandbox Verification Mode

To validate your BraneScript syntax, loops, and functional logic locally on your workstation without committing jobs to a live production cluster, use:

```bash
brane test path/to/workflow.bs
```

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
# 6. Example Workflow: Disaster Tweets Classification

This section shows a minimal, end-to-end BraneScript workflow that uses existing packages to classify “Disaster Tweets” and generate visualizations.

## 6.1 Goal
Implement and run a workflow that:

Preprocesses the Disaster Tweets training and test datasets.
Trains a Naive Bayes classifier.
Generates visualizations in an HTML file.

## 6.2 Prerequisites
Brane executable installed.
Docker installed.
Basic familiarity with running Brane commands (see Sections 5.1–5.2).

## 6.3 Building Packages and Data Assets

### 6.3.1 Build the Packages
In a terminal:

```bash
brane import epi-project/brane-disaster-tweets-example -c packages/compute/container.yml
brane import epi-project/brane-disaster-tweets-example -c packages/visualization/container.yml
brane list
```
This builds and registers:

compute: preprocessing + model training tasks
visualization: plotting and reporting tasks

### 6.3.2 Build the Data Assets
For each downloaded dataset (train and test), in the directory containing its data.yml:

```bash
brane data build ./data.yml
brane data list
```
After this, you should see identifiers such as:

nlp_train
nlp_train (training set)
nlp_test
nlp_test (test set)

## 6.4 Minimal Workflow (Compute + Visualization)
Create a workflow file (e.g., workflow.bs):

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

// Vectorize train and test together
let vectors := create_vectors(train_final, test_final);

// Train model and commit it as a reusable asset
let model := train_model(train, vectors);
commit_result("nlp_model", model);

// Inference: classify test set into a submission
let submission := create_submission(test, vectors, model);

// Visualizations bundled in an HTML file
let plot := visualization_action(
    train,
    test,
    submission
);

// Return the HTML plot so the client downloads it
return commit_result("nlp_plot", plot);
Key points:

Intermediate datasets (e.g.,
train_clean
train_clean) are not public; they are cleaned up after the workflow.
commit_result
commit_result promotes an intermediate dataset to a named, reusable asset (e.g.,
nlp_model
nlp_model,
nlp_plot
nlp_plot).
Results returned via
return
return are automatically downloaded by the client.
```

## 6.5 Local Execution

Run the workflow locally:

```bash
brane run <PATH_TO_WORKFLOW>
```
To debug, insert print statements:

```text
println("Starting preprocessing...");
println(train);
```
The final output includes a path to the downloaded nlp_plot dataset, typically containing an index.html you can open in a browser.

## 6.6 Remote Execution (Basic Pattern)

To run the same workflow on a remote Brane instance, minimally:

- Wrap tasks in an on block to choose a site (e.g., "uva" or "surf").
- Register the remote instance.
- Add the necessary certificates.
- Run with the --remote flag.

### 6.6.1 Wrap the Workflow in an on Block
Example:

```text
import compute;
import visualization;

on "uva" {
    // ... full workflow from Section 6.4
}
The on "uva" block instructs the framework to run all tasks inside on the "uva" domain.
```

### 6.6.2 Register the Instance

```bash
brane instance add brane01.lab.uvalight.net --name tutorial --use
brane instance list --show-status
```
### 6.6.3 Add Certificates
For each domain (e.g., uva, surf):

```bash
# Example for University of Amsterdam
brane certs add ./ca.pem ./client-id.pem --domain uva

# Example for SURF
brane certs add ./ca.pem ./client-id.pem --domain surf
brane certs list
```

### 6.6.4 Run the Workflow Remotely

```bash
brane run <PATH_TO_WORKFLOW> --remote
```
The resulting dataset (e.g., nlp_plot) is downloaded to your local machine, similar to local execution, but computed remotely.

This Disaster Tweets example demonstrates how BraneScript combines:

- Data assets (Section 1.3),
- Package imports and namespaces (Section 2),
- Control flow (Section 3),
- Data lineage and intermediate results (Section 4),
- Local and remote execution (Section 5),
- into a single, concise workflow for real-world NLP pipelines.
```
