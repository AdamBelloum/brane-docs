# 5. Data Policy Experts & Data Stewards Guide

This guide is tailored for **data policy experts** and **data stewards** responsible for defining regulatory constraints, configuring automated policy enforcement, and managing security tokens across a federated Brane ecosystem.

Brane is built from the ground up to operate in policy-sensitive environments—such as healthcare, finance, or cross-border research—where data cannot be centralized due to legal boundaries (e.g., GDPR) or organizational restrictions.

---

## 5.1. Policy Architecture in Brane

Brane decouples policy definition from workflow execution using two distinct layers at each domain node:

* **Policy Enforcement Point (PEP) / Policy Checker:** A native Brane stack component that intercepts all incoming workflow requests (e.g., data access, cross-domain transfers, package executions). It translates these execution events into logical queries for verification.
* **Policy Reasoner:** A pluggable formal reasoning engine (typically eFLINT) that evaluates queries sent by the PEP against a stateful set of compliance rules and facts, returning a strict **Allow** or **Deny** decision.

```

[ Scientist Workflow ] ──> [ Policy Enforcement Point (PEP) ]
│             ▲
Sends Query   │             │   Returns Decision
(Who/What)    ▼             │   (Allow / Deny)
[ eFLINT Policy Reasoner ]

```

---

## 5.2. The Policy Language (eFLINT)

Brane uses **eFLINT**, a formal domain-specific language designed specifically for compliance checking. Instead of traditional procedural programming, eFLINT operates on **Types, Facts, Rights, and Duties**.

>  **Third-Party Project Notice**
> The eFLINT language parser, command-line interpreter, and server components are developed independently as an open-source project by CWI. For comprehensive language specifications, core syntax documentation, and framework updates, visit the official repository:
> **GitHub:** [https://github.com/cwi-swat/eflint](https://github.com/cwi-swat/eflint)

### Conceptual eFLINT Syntax Pattern
When writing policies, you declare your domain structure, define relational rules, and track state transitions:

```text
// 1. Declare domain entities
Placeholder user IDENTIFIED BY string.
Placeholder dataset IDENTIFIED BY string.
Placeholder domain IDENTIFIED BY string.

// 2. Define relations and access rules
Fact may_run_dataset AGGREGATES user * dataset.
Fact personal_data AGGREGATES dataset.
Fact anonymized AGGREGATES dataset.

// 3. Define cross-domain constraints
Fact deny_transfer AGGREGATES dataset * domain * domain.
deny_transfer(D, From, To) :- personal_data(D), From != To, not anonymized(D).

```

---

## 5.3. Step-by-Step Policy Lifecycle

### Step 1: Local Offline Testing (No Token Required)

Because eFLINT is an independent third-party tool, you should always validate your policy logic locally on your machine before pushing it to a live Brane network.

1. **Install the eFLINT CLI:** Prerequisite.
Install the standalone compiler via Rust's package manager or by downloading the binary from the official repository:

```bash
cargo install eflint-cli

```

2. **Run your Policy Logic:** Terminal Execution.
Pass your policy file directly to the native interpreter to start an interactive testing session:

```bash
eflint minmax_policy.eflint

```

3. **Query the State:** Validation.
Input test assertions directly into the interactive prompt to confirm the reasoner behaves exactly as expected:

```text
?run_minmax("Adam", "minmax_dataset").

```

The terminal will output `Approved` or throw a violation error, allowing you to debug your assertions completely offline.

### Step 2: Acquire Infrastructure Trust Tokens

To deploy and update policy files on live nodes, you must authenticate using a **Policy Expert Token**.

* **Option A: You manage the infrastructure node ()**
Locate the cluster configuration files (`policy_secret.json` or `secret.json`) generated during node initialization. Run the command utility locally to issue your token:

```bash
branectl generate policy_token <user> <hostname> <number_of_days>d -s /path/to/policy_secret.json

```

This creates a `policy_token.json` configuration file in your directory.

* **Option B: The node is managed externally (e.g., Central IT / UvA Lab, ...)**
You cannot generate the token yourself. Contact your System Administrator and supply them with:

- your user identifier <`user`>, 
- your target domain <`hostname`>, 
- and your required lease duration. 

They will generate and securely transfer the `policy_token.json` file back to you.

---

### Step 3: Building & Pushing Data Configurations

To bridge these tools seamlessly, this guide demonstrates how the Brane CLI (brane) and the Administrative CLI (branectl) work in tandem during the deployment phase. 
This workflow highlights a critical separation of duties within a secure ecosystem: 

- Data Stewards and Engineers use the user-facing CLI to build and structure data metadata, 

- while Administrators utilize the control toolkit to authorize and sign the cryptographic policy tokens necessary for production enforcement.

1. **Build the Dataset Definition:** Data Steward Task.
Before applying policies to a dataset, you must package its metadata using the Brane CLI. Navigate to your dataset configuration path and build the asset:

```bash
brane build /datasets/minmax/data/data.yml

```

Note: This indexes the data structure within your local Brane workspace so it can be targeted by eFLINT rules.

2. **Generate the Policy Authorization Token:** Administrator Task Only.
Policies cannot be deployed to a live node without a cryptographic token signed by the cluster's private infrastructure keys. An administrator must run `branectl` on the control plane, referencing the node's `policy_secret.json`:

```bash
branectl generate policy_token <user> <host_name>  <number_of_days>d -s /path/to/policy_secret.json

```

This signs a token granting the user `<user>` authorization to enforce rules on the `host_name` domain for a lease duration of `<number_of_days>` days.

---

## 5.4. Common Policy Design Patterns

Use these conceptual blueprints to structure real-world governance requirements within your policy blocks:

### 5.4.1. Access Restricted by User Role

Restricts execution of specific analytical software packages to authorized personnel.

```text
fact: role(User, researcher).
fact: restricted_package(pkg_ml_train).

rule: may_run(User, Package) :-
    restricted_package(Package),
    role(User, researcher).

```

### 5.4.2. Cross-Domain Transfer Restrictions

Ensures sensitive datasets never physically leave their origin domain unless pre-processed.

```text
fact: tag(D, personal_data).
fact: origin(D, hospital_domain).

rule: deny_transfer(D, From, To) :-
    tag(D, personal_data),
    origin(D, From),
    To != From,
    not tag(D, anonymized).

```

### 5.4.3. Token-Based Compliance Claims

Leverages claims embedded inside cryptographic workflow identity tokens to grant real-time access permission.

```text
fact: token(User, Token).
fact: has_claim(Token, consent, epi_project).

rule: may_access(User, Dataset) :-
    token(User, Token),
    has_claim(Token, consent, epi_project),
    tag(Dataset, epi_data).

```
---

## 5.5. Auditing and Governance Checklist

Data policies remain incomplete without ongoing compliance verification. As a steward, regularly verify:

* **Global Audit Alignment:** Cross-domain tracking records logs of every multi-party workflow submission and data movement event.

* **Local Execution Logs:** Individual worker domains preserve strict event timelines showing exactly who called what dataset, which package ran, and the underlying reasons behind PEP approvals or denials.

* **Obligation Loops:** Ensure any required data transformations (e.g., mandatory anonymization pipelines) are successfully registered as `done(...)` in the reasoner state before downstream operations run.

```

```
