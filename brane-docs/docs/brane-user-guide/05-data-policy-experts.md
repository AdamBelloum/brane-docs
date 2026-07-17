## 5. Data Policy Experts / Data Stewards: Policies and the Checker

This section is for **data policy experts** and **data stewards** who are responsible for:

- Defining data and network policies,
- Configuring Brane’s **policy checker** and **reasoner**,
- Managing tokens and secrets used to enforce policies.

Brane is designed to operate in **policy-sensitive environments**, especially where:

- Data is distributed across domains,
- Regulations (e.g., GDPR) require strict control over data access and movement,
- Different organizations own and control their datasets.

---

### 5.1. Policy concepts in Brane

Brane distinguishes two broad classes of policies:

- **Data policies**  
  Rules governing:
  - Who can read/write a dataset,
  - Under what conditions data can move between domains,
  - How data is processed (e.g., anonymization requirements).

- **Network / infrastructure policies**  
  Rules about:
  - Which services can communicate and how,
  - Which domains can connect to which others,
  - Which nodes are allowed to perform certain tasks (e.g., executing particular packages).

These policies are enforced by:

- A **policy checker** (Policy Enforcement Point, PEP) in each domain,
- A **reasoner** (typically eFLINT) that evaluates policy rules.

---

### 5.2. Policy checker and reasoner (e.g., eFLINT)

The **policy checker** is a component that:

- Sits between Brane’s execution logic and the domain’s data/backends,
- Receives requests (e.g., “may workflow X read dataset Y from domain Z?”),
- Consults the **reasoner** to determine whether the request complies with policy.

#### 5.2.1. Reasoner (eFLINT)

In many Brane deployments, the reasoner is **eFLINT**, a formal policy engine:

- Policies are expressed in a logical language (eFLINT rules),
- The reasoner maintains a state of facts and rules,
- Each policy query is evaluated against these rules.

Examples of what eFLINT can express (conceptually):

- “Datasets containing personal data may not leave the origin domain unless pseudonymized.”
- “Only members of group A may run package B on dataset C.”
- “Domain D may not accept incoming connections from unspecified domains.”

The policy expert writes eFLINT rules and manages the reasoner’s state.

#### 5.2.2. Policy Enforcement Point (PEP)

The PEP (checker):

- Receives requests from Brane’s planner and delegates for data access and movement,
- Translates them into queries for the reasoner,
- Returns **allow/deny** decisions, possibly with additional obligations (e.g., “must anonymize first”).

The PEP is integrated into the worker domain’s Brane stack so that:

- All data access/movement operations must pass through it,
- It can enforce policy consistently across workflows and packages.

---

### 5.3. Tokens and secrets

Policy enforcement often requires **tokens and secrets** to authenticate and authorize actions:

- Tokens may represent:
  - User identities,
  - Role memberships,
  - External policy decisions.

- Secrets may include:
  - Keys used by the reasoner or PEP,
  - Credentials for secured datasets or services.

Brane’s policy infrastructure may:

- Use tokens attached to workflows or API calls,
- Forward tokens to the reasoner to make decisions based on user roles and attributes.

As a policy expert/data steward, you:

- Manage token formats (e.g., JWT or custom),
- Ensure that tokens contain the necessary attributes for policy decisions,
- Coordinate with administrators to secure secret storage and distribution.

---

### 5.4. Defining and managing policies

#### 5.4.1. Policy scope and granularity

Policies in Brane can be defined at different levels:

- **Dataset-level**  
  e.g., “Dataset `patients` may only be used in anonymized form by external domains.”

- **Package-level**  
  e.g., “Package `ml.train` may not be invoked on raw personal data.”

- **Domain-level**  
  e.g., “Domain `research` may access certain datasets but not others; domain `prod` may not export datasets.” [1]

Policy experts should:

- Identify sensitive datasets and functions,
- Decide which operations are permitted across domains,
- Encode these rules in the reasoner (eFLINT). 

#### 5.4.2. Policy authoring workflow

Typical workflow for policy authoring:

1. **Identify policy requirements**  
   - Regulatory (e.g., GDPR),
   - Organizational (internal guidelines),
   - Project-specific (e.g., EPI project care pathways).

2. **Model requirements as logical rules**  
   - Translate to eFLINT or similar language:
     - Entities (datasets, domains, roles),
     - Actions (read, write, move, process),
     - Conditions and constraints. 

3. **Load rules into the reasoner**  
   - Configure the reasoner with policy modules,
   - Initialize facts (e.g., which dataset belongs to which domain).

4. **Integrate with PEP**  
   - Ensure the PEP sends appropriate queries,
   - Confirm that decisions are correctly enforced:
     - Allow/deny access,
     - Trigger obligations (e.g., anonymization step).

#### 5.4.3. Testing policies

Before full deployment:

- Use **test workflows** that exercise policy-relevant operations:
  - Accessing protected datasets,
  - Attempting cross-domain transfers,
  - Running packages with different user roles.

- Verify that:
  - Allowed operations succeed,
  - Disallowed operations fail with clear messages,
  - Obligations (such as required preprocessing) are applied correctly.

You may also have dedicated CLI or admin tools for:

- Sending test queries to the reasoner/PEP,
- Inspecting reasoner state (facts and rules),
- Logging decisions to audit trails.

---

### 5.5. Policy-aware workflows

Policy experts collaborate with scientists and package developers to design **policy-aware workflows**.

#### 5.5.1. Communicating policy constraints

Scientists should know:

- Which datasets are sensitive,
- Which domains they can use for specific data,
- Which packages are restricted or require special conditions.

Package developers should know:

- When their packages are expected to:
  - Anonymize or transform data before export,
  - Only run on certain domains,
  - Log additional metadata for audit purposes.

#### 5.5.2. Handling policy violations

When a workflow violates a policy:

- The PEP denies the operation,
- The workflow may fail or fallback to an alternative path (if defined).

Policy experts:

- Review logs and reasoner decisions,
- Adjust rules if needed,
- Help scientists re-design workflows to comply with policies.

---

### 5.6. Audit and governance

Policies are tightly connected to **audit and governance**.

Brane maintains:

- **Global audit logs** – cross-domain workflow submissions, task assignments, data movements.
- **Local audit logs** – domain-specific activities (data access, local execution).

Policy experts/data stewards use these logs to:

- Demonstrate compliance (e.g., to regulators),
- Investigate incidents or suspicious activity,
- Refine policies based on observed workflow patterns.

They may also:

- Align audit schemas with external governance frameworks,
- Integrate Brane logs with organizational SIEM or audit systems.

---

### 5.7. Summary


For data policy experts and data stewards, Brane provides:

- A **policy checker (PEP)** integrated with workflow execution,
- A **reasoner** (e.g., eFLINT) for expressing and evaluating rich policy rules,
- Mechanisms for passing **tokens and secrets** into policy decisions,
- Integration with audit logs for governance and compliance. 

Your responsibilities include:

- Defining and maintaining policies that control data access and movement,
- Configuring and monitoring the checker and reasoner,
- Collaborating with scientists, package developers, system engineers, and administrators to ensure that workflows respect both technical and regulatory constraints.

### 5.8. Policy Quick Reference (Conceptual)

This section summarizes typical policy patterns that Brane deployments address using a reasoner such as **eFLINT** and the policy checker (PEP). Use it as a **design aid**, not exact syntax.

> **Note:** The examples below are **pseudo‑rules**, inspired by eFLINT‑style thinking but not guaranteed to match your deployed syntax. Adapt them to your actual policy language and engine.


#### 5.8.1. Basic structure of a policy rule

Most rules follow this structure:

- **Facts** – statements about the world:
  - “Dataset `patients` belongs to domain `hospital`.”
  - “User `alice` has role `researcher`.”
  - “Package `ml.train` is sensitive.”

- **Rules** – implications that define allowed/denied actions:
  - “If a user is a researcher and a dataset is anonymized, then reading is allowed.”
  - “If a dataset contains personal data and destination domain is external, then transfer is denied.”

Conceptually:
```text
fact: owns(hospital, patients).
fact: role(alice, researcher).
fact: tag(patients, personal_data).

rule: may_read(User, Dataset) :-
    role(User, researcher),
    tag(Dataset, anonymized).

rule: deny_transfer(Dataset, FromDomain, ToDomain) :-
    tag(Dataset, personal_data),
    external(ToDomain),
    not same_domain(FromDomain, ToDomain).
```

#### 5.8.2. “Only anonymized data may leave the origin domain”

Goal: Prevent raw personal data from leaving its origin domain.

Conceptual pattern:

```text
fact: tag(D, personal_data) :- ...         # D is a personal dataset
fact: origin(D, OriginDomain).

rule: deny_transfer(D, From, To) :-
    tag(D, personal_data),
    origin(D, OriginDomain),
    To != OriginDomain,
    not tag(D, anonymized).

rule: may_transfer(D, From, To) :-
    tag(D, personal_data),
    origin(D, OriginDomain),
    To != OriginDomain,
    tag(D, anonymized).
```
- Interpretation:

If a dataset is personal and the destination domain is different from its origin, transfer is denied unless the dataset is tagged as anonymized.

#### 5.8.3. “Only specific roles may run certain packages”

Goal: Restrict sensitive packages (e.g., training models on sensitive data) to certain user roles.

Conceptual pattern:
```text
fact: role(User, researcher).
fact: role(User, admin).
fact: restricted_package(pkg_ml_train).

rule: may_run(User, Package) :-
    restricted_package(Package),
    role(User, researcher).

rule: may_run(User, Package) :-
    restricted_package(Package),
    role(User, admin).

rule: deny_run(User, Package) :-
    restricted_package(Package),
    not (role(User, researcher); role(User, admin)).
```
- Interpretation:
Only users with roles researcher or admin may run pkg_ml_train.
Other users are denied.

#### 5.8.4. “Domain‑level access control for datasets”

Goal: Restrict datasets to specific domains.

Conceptual pattern:
```text
fact: owns(hospital, patients).
fact: allowed_domain(patients, research).

rule: may_access(Domain, Dataset) :-
    owns(Domain, Dataset).

rule: may_access(Domain, Dataset) :-
    allowed_domain(Dataset, Domain).

rule: deny_access(Domain, Dataset) :-
    not may_access(Domain, Dataset).
```
- Interpretation:

The owning domain may access the dataset.
Additionally, a dataset may list other allowed_domains.
All other domains are denied.

#### 5.8.5. “Obligations: must preprocess before access”

Goal: Require preprocessing (e.g., anonymization) before a dataset is used by a given package or domain.

Conceptual pattern:
```text
fact: tag(D, personal_data).
fact: preprocess_step(D, anonymize_step).

rule: obligation(User, D, anonymize_step) :-
    tag(D, personal_data),
    may_run(User, some_sensitive_package).

rule: may_access_after_preprocess(Domain, D) :-
    obligation(User, D, anonymize_step),
    done(User, D, anonymize_step).
```

- Interpretation:

For some combinations of user/package/dataset, there is an obligation to run anonymize_step.
Access is permitted only after the obligation is satisfied (recorded as done(...)).
The PEP can use obligations to instruct Brane to:

Insert preprocessing steps into workflows, or
Deny access until required preprocessing actions have been performed.


#### 5.8.6. Token‑based decisions
Goal: Use tokens (e.g., JWTs) to drive decisions.

Tokens might encode:

- User identity,
- Roles,
- Project memberships,
- Consent flags.

Conceptual pattern:

```text
fact: token(User, Token).
fact: has_claim(Token, role, researcher).
fact: has_claim(Token, consent, epi_project).

rule: may_access(User, Dataset) :-
    token(User, Token),
    has_claim(Token, role, researcher),
    has_claim(Token, consent, epi_project),
    tag(Dataset, epi_data).
```
Interpretation:
A user may access epi_data datasets only if their token has the right role and consent claims.

### 5.8.7. Practical checklist for policy experts
When designing policies:

Identify entities:
- Datasets, domains, packages, users, roles.
Define facts:
- Ownership, sensitivity tags (personal, anonymized), origin domains.
Write rules:
- may_* for allowed operations,
- deny_* for explicit prohibitions,
- obligation rules for required preprocessing.
Test with sample workflows:
- Access from allowed and disallowed domains,
- Runs from different user roles,
- Transfers of sensitive datasets.
- Integrate with Brane’s PEP:
- Ensure requests (e.g., data access, transfer, package run) are correctly translated into reasoner queries.
- Verify that decisions and obligations propagate back into workflow execution.
- This quick reference is intended to guide policy modeling; always consult your actual reasoner’s documentation (e.g., eFLINT reference) for precise syntax and semantics.
