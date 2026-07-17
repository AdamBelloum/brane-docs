
---

# 4. Part 2: Disaster Tweets workflow

In this part, you:

- Build two existing packages (`compute` and `visualization`) from a Git repository,
- Register training and test datasets in Brane,
- Write a BraneScript workflow that:
  - Cleans, tokenizes, and vectorizes tweets,
  - Trains a Naive Bayes classifier,
  - Generates visualizations,
- Run the workflow locally (and optionally remotely).

---

## 4.1. Background

You now take the role of a **domain scientist** using an existing package.

The pipeline:

- Implements a classifier that predicts whether a tweet refers to a natural disaster (1) or not (0).
- Uses preprocessed tweets as input to a Naive Bayes classifier.
- Includes visualizations to explore:
  - Dataset properties,
  - Model behaviour.

This tutorial uses a version of the package created by students (originally for a Kaggle challenge) and adapted to Brane.

---

## 4.2. Objective

You will:

1. Build two packages:
   - `compute` – preprocessing and training functions.
   - `visualization` – plotting and HTML generation.

2. Register two datasets:
   - Training dataset.
   - Test dataset.

3. Write a workflow that:
   - Cleans and preprocesses both datasets,
   - Vectorizes them,
   - Trains the classifier,
   - Generates visualizations and commits them as datasets.

---

## 4.3. Building packages from Git

Brane can import package definitions from Git repositories.

Assuming the example repository `epi-project/brane-disaster-tweets-example`, run:

```bash
# Syntax to be validated later
brane import epi-project/brane-disaster-tweets-example -c packages/compute/container.yml
brane import epi-project/brane-disaster-tweets-example -c packages/visualization/container.yml
```
Conceptual behaviour:

brane import builds packages from a remote Git repo.
-c specifies which container.yml in the repo to use:
packages/compute/container.yml,
packages/visualization/container.yml.
After completion, verify:

```bash
# Syntax to be validated later
brane list
```
You should see compute and visualization packages available locally.

## 4.4. Registering datasets
The training and test datasets are shipped as pre‑packaged archives containing:

data.yml – dataset metadata,
data/dataset.csv – actual data.
Unpack the training dataset archive, then run:

```bash
# Syntax to be validated later
brane data build ./data.yml
```
Repeat for the test dataset.

Check:

```bash
# Syntax to be validated later
brane data list
```
You should see dataset identifiers such as nlp_train and nlp_test.

By default, Brane links the dataset files rather than copying them. If you plan to delete original files, use a --no-links flag (or equivalent) so Brane copies the data instead.

##  4.5. Writing the workflow – compute pipeline
Create workflow.bs and start by importing packages and referencing datasets.

```branescript
import compute;
import visualization;

// Refer to training and test datasets
let train := new Data{ name := "nlp_train" };
let test  := new Data{ name := "nlp_test" };
```
Notes:

- Data{ name := "..." } constructs a dataset handle for Brane.
- The handle references a dataset by its registered name.
- The actual data is not directly accessible from BraneScript; the handle tells Brane which dataset to attach to package tasks.

### 4.5.1. Cleaning datasets


```branescript
// Step 1: clean training and test datasets
let train_clean := clean(train);
let test_clean  := clean(test);
```
- clean() is a function in the compute package.
- It outputs new datasets (intermediate results) with cleaned text.
- These intermediate datasets are not yet public; they are used only within the workflow.

### 4.5.2. Tokenizing and removing stopwords

```branescript
// Step 2: tokenize
let train_final := tokenize(train_clean);
let test_final  := tokenize(test_clean);

// Step 3: remove stopwords
train_final := remove_stopwords(train_final);
test_final  := remove_stopwords(test_final);
```
- tokenize() splits text into tokens.
- remove_stopwords() filters out common, uninformative words.
- We reuse train_final and test_final variables instead of creating new ones.

### 4.5.3. Vectorizing datasets
The package vectorizes training and test datasets together:

```branescript
// Step 4: vectorize datasets (training + test together)
let vectors := create_vectors(train_final, test_final);
```
- vectors represents vectorized forms of both datasets.
- Subsequent steps use these vectorized forms for training and inference.

### 4.5.4. Training the classifier

```branescript
// Step 5: train model
let model := train_model(train, vectors);
commit_result("nlp_model", model);
train_model(train, vectors) trains the Naive Bayes classifier:
```
- Uses the original training dataset (labels),
- Uses the vectorized features.
commit_result("nlp_model", model):
- Promotes the intermediate result model to a publicly available dataset with identifier nlp_model.
- This allows future workflows to reuse the trained model without retraining.

Returning models as datasets:

- Aligns with typical library behaviour (models stored as files),
- Enables policy control over where models can be stored and used.

## 4.6. Writing the workflow – visualization
We now add inference and visualizations.

### 4.6.1. Creating a submission (classifying test set)

```branescript
// Classify test set and create "submission"
let submission := create_submission(test, vectors, model);
create_submission() uses:
test dataset,
vectors (vectorized features),
```
- model (trained classifier).
- Output: a dataset mapping tweet IDs to predictions (0 or 1).

### 4.6.2. Generating plots in a single HTML report
The simplest way to produce visualizations is to use the generic visualization_action() function from the visualization package:

```branescript
// Generate combined visualizations as an HTML report
let plot := visualization_action(
    train,
    test,
    submission
);

// Commit and return the report dataset
return commit_result("nlp_plot", plot);
```
visualization_action():
- Consumes the training dataset, test dataset, and submission,
- Produces an HTML report with multiple plots and metrics.
commit_result("nlp_plot", plot):
- Makes the report available as a dataset nlp_plot.
return commit_result(...):
- Returns the dataset to the client, which can then download and inspect the HTML report.
The visualization package usually also exposes individual plot functions; you can experiment by calling them separately and extending the workflow.

## 4.7. Running the workflow locally
Run:

```bash
# Syntax to be validated later
brane run ./workflow.bs
```
During execution, you can add println() calls to trace progress, e.g.:

```branescript
println("Cleaning dataset...");
let train_clean := clean(train);
...
println("Training model...");
let model := train_model(train, vectors);
...
println("Generating interactive plot...");
let plot := visualization_action(train, test, submission);
```
Console output may look like:

```text
Cleaning dataset...
Tokenizing dataset...
Removing stopwords from dataset...
Performing feature vectorization...
Training model...
Generating bigrams...
Classifying tweets...
Generating interactive plot...
```
Workflow returned value 'nlp_plot'
(It’s available under '<path>/nlp_plot/data')
Open the HTML report (index.html or equivalent) in a browser (e.g., Firefox) to inspect the plots and metrics.

## 4.8. Remote execution (conceptual)
A more advanced part of the tutorial runs the same workflow on a remote Brane instance (e.g., with worker nodes at UvA and SURF).

Key steps (conceptual):

Annotate workflow with on blocks to choose domains:
```branescript
on "uva" {
    // whole pipeline here
}
```
Register the remote instance with the Brane CLI:

```bash
# Syntax to be validated later
brane instance add brane01.lab.uvalight.net --name tutorial --use
```
Add client certificates for domains (e.g., uva, surf):

```bash
brane certs add ./ca.pem ./client-id.pem --domain uva
brane certs add ./ca.pem ./client-id.pem --domain surf
```
Run workflow remotely:
```bash
brane run ./workflow.bs --remote
```
Behavior:

- Workflow runs on the remote instance (domains uva or surf).
- Final datasets (e.g., nlp_plot) are downloaded back to your client.
- In current versions, on structures and remote execution semantics may have changed; treat this section as conceptual. We will update concrete commands and annotations based on your deployment notes later.

## 4.9. Summary
In Part 2 you:

- Imported and built realistic packages from Git,
- Registered training and test datasets,
- Wrote a multi‑step BraneScript workflow for:
- Preprocessing,
- Vectorization,
- Model training,
- Visualization,
- Ran the workflow locally (and conceptually remotely).

This tutorial demonstrates how Brane handles real‑world, data‑heavy pipelines and how scientists can work with datasets, intermediate results, and multi‑domain execution.