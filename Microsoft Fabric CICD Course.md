# Microsoft Fabric CI/CD with Azure DevOps YAML

## Basic-to-Master Training Course

> **Audience:** Fabric developers, data engineers, analytics engineers, BI developers, and DevOps engineers  
> **Level:** Absolute beginner to advanced; no YAML knowledge required  
> **Estimated duration:** 12-13 weeks, 6-8 hours per week  
> **Primary tools:** Microsoft Fabric, Azure Repos, Azure Pipelines YAML, Microsoft Entra ID, Azure Key Vault, and `fabric-cicd`  
> **Reference date:** September 22, 2026

Microsoft Fabric evolves frequently. Before applying a pattern to production, verify that every item type you use supports the required Git, REST API, service principal, deployment pipeline, variable library, and auto-binding capabilities.

---

## 1. What You Will Be Able to Do

By the end of this course, you will be able to:

1. Explain CI, continuous delivery, and continuous deployment.
2. Read and write Azure Pipelines YAML.
3. Store Fabric item definitions in Azure Repos.
4. Design Dev, UAT, and Production Fabric environments.
5. Validate pull requests before changes are merged.
6. Deploy Fabric items with `fabric-cicd` or Fabric deployment APIs.
7. Replace environment-specific values safely.
8. Protect credentials with service principals and Azure Key Vault.
9. Configure approvals, branch controls, and release checks.
10. Test schemas, dependencies, data quality, and operational readiness.
11. Diagnose failed releases and perform controlled rollback.
12. Design a reusable enterprise CI/CD platform for multiple Fabric solutions.

---

## 2. The Course Project

You will incrementally build a solution named **Sales Analytics**.

### Fabric components

- `SalesLakehouse`
- `IngestSales.Notebook`
- `TransformSales.Notebook`
- `SalesOrchestration.DataPipeline`
- `SalesModel.SemanticModel`
- `SalesReport.Report`
- `environment_settings.VariableLibrary`

### Environments

| Environment | Fabric workspace | Purpose | Deployment policy |
|---|---|---|---|
| Development | `SalesAnalytics-Dev` | Authoring and developer testing | Frequent changes |
| UAT | `SalesAnalytics-UAT` | Integrated and business testing | Approval required |
| Production | `SalesAnalytics-Prod` | Business use | Approval and release checks required |

### Target delivery flow

```text
Feature branch
    |
    v
Pull request
    |
    +--> CI: validate YAML, item definitions, policy, and tests
    |
    v
Main branch / immutable release
    |
    v
Deploy to UAT --> validate --> approve
    |
    v
Deploy the same release to Production --> smoke test --> monitor
```

The important principle is **build once, promote the same reviewed content, and change only environment configuration**.

---

## 3. Recommended Study Method

For each module:

1. Read the lesson.
2. reproduce the example in a non-production project;
3. complete the lab without copying the final solution;
4. answer the quiz;
5. explain the design to another person; and
6. record one failure you encountered and how you diagnosed it.

Do not practice deployment automation against Production. Use disposable or dedicated training workspaces.

If you have never used YAML, complete every exercise in Module 3 before copying the later pipeline examples. Module 3 starts with plain text and assumes no programming or YAML experience.

---

# Learning Path

| Phase | Modules | Outcome |
|---|---|---|
| Foundation | 1-3 | Understand CI/CD, Git, YAML, and Azure Pipelines |
| Fabric practitioner | 4-6 | Version Fabric items and manage environment configuration |
| Deployment engineer | 7-9 | Build secure CI and multi-stage CD pipelines |
| Advanced engineer | 10-12 | Test, observe, recover, and scale the platform |
| Mastery | Capstone | Deliver and defend a production-grade implementation |

---

# Module 1 - CI/CD Foundations

## Objectives

- Explain the problem CI/CD solves.
- Distinguish CI from continuous delivery and continuous deployment.
- Identify the stages of a release pipeline.
- Use the core terminology correctly.

## 1.1 Why CI/CD Exists

Manual releases often create these problems:

- the process is not repeatable;
- deployment depends on one person's knowledge;
- changes are made directly in Production;
- environment settings are copied incorrectly;
- there is no reliable record of what was released;
- testing happens late;
- rollback means manually reconstructing an older state; and
- a release works from one machine but not from another.

CI/CD converts release knowledge into versioned, repeatable automation.

## 1.2 CI, Continuous Delivery, and Continuous Deployment

| Term | Meaning | Fabric example |
|---|---|---|
| Continuous Integration | Frequently merge changes and automatically validate them | A PR validates Fabric item definitions and notebook rules |
| Continuous Delivery | Keep a release deployable, but require a human decision for Production | A UAT-approved release waits for a Production approver |
| Continuous Deployment | Automatically deploy every qualifying change without manual approval | A low-risk internal Development workspace updates after every merge |

For most governed Fabric estates, **continuous delivery** is safer than automatic Production deployment.

## 1.3 Universal Pipeline Flow

```text
Commit -> Validate -> Package -> Publish -> Deploy -> Verify -> Monitor
```

Fabric item definitions might not be "compiled" like application code. The build phase still has value: it can validate JSON, inspect item metadata, run policy checks, calculate the release scope, and publish an immutable pipeline artifact.

## 1.4 Core Glossary

| Term | Definition |
|---|---|
| Pipeline | Automated workflow that performs CI or CD |
| Trigger | Event that starts a pipeline |
| Stage | Major boundary such as Validate, UAT, or Production |
| Job | Unit of work assigned to an agent |
| Step | Ordered action in a job |
| Task | Packaged Azure Pipelines action |
| Agent | Machine that executes a job |
| Pool | Collection of agents |
| Artifact | Versioned output promoted by later stages |
| Environment | Deployment target with history and checks |
| Gate/check | Policy that must pass before a resource is used |
| Release | A specific, traceable version promoted to an environment |

## Lab 1 - Map a Manual Release

Write down how one Fabric change currently reaches Production.

Create a table with:

- actor;
- action;
- input;
- output;
- validation;
- approval;
- evidence; and
- rollback.

Mark every step that is manual, unrecorded, or person-dependent. Convert the three highest-risk steps into candidate automation tasks.

## Quiz 1

1. What is the primary goal of CI?
2. Why is continuous delivery usually more appropriate than continuous deployment for governed Production workspaces?
3. What should be identical between UAT and Production promotion?
4. What is the difference between a stage and a step?

---

# Module 2 - Git and Azure Repos for Fabric

## Objectives

- Use commits, branches, pull requests, and tags.
- Explain why Git is not the same as a Fabric workspace.
- Design a safe branch strategy.
- Apply branch policies.

## 2.1 Git Mental Model

```text
Working copy -> Commit -> Branch -> Pull request -> Protected branch -> Release tag
```

A commit is a snapshot with identity, author, time, and parent history. A branch is a movable reference to a commit. A tag should identify an immutable release.

## 2.2 Fabric Workspace versus Repository

| Fabric workspace | Git repository |
|---|---|
| Live service state | Version history |
| Can contain data and runtime configuration | Contains item definitions and automation code |
| Users can edit through the portal | Developers review text changes |
| May drift through direct edits | Provides auditable commits |

Treat the repository as the source of truth for deployable definitions. Data, credentials, tokens, generated logs, and local configuration do not belong in Git.

## 2.3 Recommended Beginner Branch Strategy

Use:

- short-lived `feature/*` branches;
- a protected `main` branch;
- release tags such as `fabric-sales-v1.3.0`.

For teams following the Microsoft `fabric-cicd` branch-promotion tutorial, long-lived `dev`, `test`, and `prod` branches can record environment state. Learn that model, but choose it deliberately: extra long-lived branches increase merge and drift management.

## 2.4 Minimum Branch Policies

Protect `main` with:

- pull requests required;
- minimum reviewers;
- self-approval restricted where required;
- successful CI build required;
- comment resolution required;
- linked work item required if your governance model uses one;
- direct pushes restricted; and
- force pushes prohibited.

Production deployment should also use an Azure DevOps environment **branch control check** that permits only an approved fully qualified branch such as `refs/heads/main`.

## 2.5 Commit and PR Quality

A useful PR answers:

- What changed?
- Why did it change?
- Which Fabric items are affected?
- Which dependencies could break?
- How was it tested?
- Does configuration change?
- How can the release be rolled back?

## Lab 2 - Build the Repository

Create this structure:

```text
sales-analytics/
|-- fabric/
|   |-- SalesLakehouse.Lakehouse/
|   |-- IngestSales.Notebook/
|   |-- SalesOrchestration.DataPipeline/
|   `-- environment_settings.VariableLibrary/
|-- pipelines/
|   |-- templates/
|   `-- azure-pipelines.yml
|-- scripts/
|-- tests/
|-- requirements.txt
`-- README.md
```

Create a feature branch, change a harmless notebook comment, commit it, open a PR, and inspect the diff before merging.

## Quiz 2

1. Why should a Fabric workspace not be treated as the only source of truth?
2. What is the advantage of a short-lived feature branch?
3. Why should Production accept releases only from protected branches?
4. When would a release tag be more useful than a branch name?

---

# Module 3 - YAML from Zero and Azure Pipelines Anatomy

## Objectives

- Understand what YAML is and why Azure Pipelines uses it.
- Write values, lists, maps, comments, and multiline text.
- Read YAML indentation, mappings, and sequences.
- Translate a plain-English workflow into YAML.
- Explain triggers, parameters, variables, stages, jobs, and steps.
- Distinguish compile-time and runtime expressions.
- Create a valid starter pipeline.

## 3.1 What Is YAML?

YAML is a human-readable way to describe structured information. Azure DevOps reads a YAML file as a set of instructions for a pipeline.

Compare plain English:

```text
Use an Ubuntu computer.
Run a step named Say hello.
The step prints Hello.
```

With YAML:

```yaml
pool:
  vmImage: ubuntu-latest

steps:
  - script: echo "Hello"
    displayName: Say hello
```

YAML is not a programming language. It describes **what** the pipeline contains. Tasks and scripts perform the actual work.

## 3.2 The Three Building Blocks

Almost every YAML file is built from:

1. scalar values;
2. mappings; and
3. sequences.

### Scalar values

A scalar is one value:

```yaml
name: Fabric-CI
enabled: true
retryCount: 3
environment: uat
```

Here:

- `Fabric-CI` and `uat` are strings;
- `true` is a Boolean; and
- `3` is a number.

Quote a value when it could be interpreted incorrectly:

```yaml
version: "1.0"
answer: "yes"
dateCode: "2026-09-22"
```

### Mappings: named properties

A mapping is a group of `key: value` pairs:

```yaml
environment:
  name: uat
  requiresApproval: true
```

Read this as:

> The environment has a name of `uat` and requires approval.

`name` and `requiresApproval` belong to `environment` because they are indented beneath it.

### Sequences: ordered lists

A sequence uses a dash for every item:

```yaml
itemTypes:
  - Notebook
  - Lakehouse
  - DataPipeline
```

Read this as:

> `itemTypes` is a list containing Notebook, Lakehouse, and DataPipeline.

### Lists of mappings

Azure Pipelines frequently uses a list where every entry has several properties:

```yaml
steps:
  - script: echo "Validate"
    displayName: Validate source

  - script: echo "Package"
    displayName: Package release
```

`steps` is a list. Each dash starts a new step. Each step contains a `script` and a `displayName`.

## 3.3 Indentation: The Most Important Rule

Indentation defines ownership and nesting.

Correct:

```yaml
pool:
  vmImage: ubuntu-latest
```

Incorrect:

```yaml
pool:
vmImage: ubuntu-latest
```

In the incorrect example, `vmImage` is no longer inside `pool`.

Use these rules:

- use spaces, never tabs;
- use two spaces for each level in this course;
- align sibling properties at the same column;
- indent child properties exactly one level;
- put one space after `:`; and
- put one space after `-`.

Visualize nesting as boxes:

```text
stages
`-- stage: Validate
    `-- jobs
        `-- job: ValidateDefinitions
            `-- steps
                `-- script: echo "Validate"
```

The matching YAML is:

```yaml
stages:
  - stage: Validate
    jobs:
      - job: ValidateDefinitions
        steps:
          - script: echo "Validate"
```

### Indentation exercise

Fix this YAML:

```yaml
stages:
- stage: Validate
jobs:
- job: Check
steps:
- script: echo "Checking"
```

Correct answer:

```yaml
stages:
  - stage: Validate
    jobs:
      - job: Check
        steps:
          - script: echo "Checking"
```

## 3.4 Comments and Multiline Scripts

Comments begin with `#`:

```yaml
# Run only on the protected main branch
trigger:
  - main
```

Use `|` for a multiline script. YAML preserves the line breaks:

```yaml
steps:
  - pwsh: |
      Write-Host "First command"
      Write-Host "Second command"
    displayName: Run two PowerShell commands
```

Use `>` or `>-` to fold several YAML lines into one command:

```yaml
steps:
  - script: >-
      python scripts/deploy_fabric.py
      --environment uat
      --repository-directory fabric
    displayName: Deploy to UAT
```

For a beginner:

- use `|` when the lines are separate commands;
- use `>-` when one long command is wrapped for readability.

## 3.5 Read YAML from the Outside In

Do not try to understand a large pipeline one line at a time. Read it in layers:

1. Find the top-level sections: `trigger`, `pool`, `variables`, `stages`.
2. List the stage names.
3. For one stage, list its jobs.
4. For one job, list its steps.
5. Only then inspect task inputs and scripts.

Example:

```yaml
trigger:
  - main

pool:
  vmImage: ubuntu-latest

stages:
  - stage: Validate
    jobs:
      - job: CheckSource
        steps:
          - script: echo "Checking Fabric definitions"
            displayName: Check source

  - stage: Package
    dependsOn: Validate
    jobs:
      - job: CreateArtifact
        steps:
          - script: echo "Creating release"
            displayName: Create release
```

Read it as:

- a commit to `main` starts the pipeline;
- an Ubuntu agent executes it;
- the `Validate` stage runs first;
- `CheckSource` is the job in `Validate`;
- its only step prints a message;
- `Package` waits for `Validate`; and
- `CreateArtifact` is the job in `Package`.

## 3.6 Common Beginner Errors

### Tabs instead of spaces

Tabs can make valid-looking YAML fail. Configure the editor to insert spaces.

### A missing colon

Incorrect:

```yaml
displayName Validate source
```

Correct:

```yaml
displayName: Validate source
```

### A missing dash in a list

Incorrect:

```yaml
steps:
  script: echo "One"
  script: echo "Two"
```

Correct:

```yaml
steps:
  - script: echo "One"
  - script: echo "Two"
```

### Misaligned sibling properties

Incorrect:

```yaml
steps:
  - script: echo "Hello"
      displayName: Say hello
```

Correct:

```yaml
steps:
  - script: echo "Hello"
    displayName: Say hello
```

### Using a property in the wrong place

Syntactically valid YAML can still be invalid Azure Pipelines YAML. For example, Azure DevOps expects `steps` inside a job, not directly inside `stages`. YAML validation and Azure Pipelines schema validation are different checks.

### Repeating a mapping key

Avoid this:

```yaml
variables:
  environment: uat
  environment: prod
```

Use one value or a sequence/design that represents the real intent.

## 3.7 Azure Pipelines Hierarchy

```text
pipeline
  stages
    jobs
      steps
        tasks or scripts
```

Definitions:

| Level | Simple meaning | Example |
|---|---|---|
| Pipeline | The entire automated workflow | Fabric CI/CD |
| Stage | A major phase | Validate, UAT, Production |
| Job | Work assigned to one agent | Validate definitions |
| Step | One ordered action | Run a Python validator |
| Task | A packaged Azure DevOps action | `UsePythonVersion@0` |
| Script | Commands you write | PowerShell, Bash, or Python command |

## 3.8 Your First Azure Pipeline

Start with the smallest possible pipeline:

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

steps:
  - script: echo "My first Azure Pipeline"
    displayName: Say hello
```

Line-by-line:

| YAML | Meaning |
|---|---|
| `trigger: none` | Do not start automatically |
| `pool:` | Select the agent configuration |
| `vmImage: ubuntu-latest` | Use a Microsoft-hosted Ubuntu agent |
| `steps:` | Begin the ordered list of actions |
| `- script:` | Start a script step |
| `displayName:` | Friendly name shown in the pipeline UI |

Next, add a variable:

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

variables:
  message: Learning YAML

steps:
  - script: echo "$(message)"
    displayName: Print a variable
```

`$(message)` asks Azure Pipelines to replace the expression with the value of the `message` variable.

## 3.9 Minimal Fabric CI Pipeline

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - fabric/**
      - pipelines/**
      - scripts/**

pr:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

stages:
  - stage: Validate
    displayName: Validate Fabric source
    jobs:
      - job: ValidateDefinitions
        steps:
          - checkout: self
          - pwsh: |
              $platformFiles = Get-ChildItem fabric -Recurse -Force -Filter ".platform"
              if ($platformFiles.Count -eq 0) {
                throw "No Fabric item definitions were found."
              }
              Write-Host "Found $($platformFiles.Count) Fabric item definitions."
            displayName: Discover Fabric items
```

Before continuing, identify:

- the four top-level sections;
- the included branch;
- the three included paths;
- the stage name;
- the job name;
- the two steps; and
- the multiline PowerShell block.

## 3.10 Trigger versus PR Validation

- `trigger` controls CI runs after commits reach included branches.
- `pr` describes pull-request validation where supported.
- In Azure Repos, configure the required pipeline as a **build validation branch policy** so PR completion depends on it.
- Use path filters to avoid unrelated runs, but do not accidentally exclude shared templates or validation scripts.

## 3.11 Parameters and Variables

| Feature | Evaluated | Best use |
|---|---|---|
| Parameter | Template parsing / queue time | Type-safe structural choices |
| Static variable | Pipeline processing | Reusable string values |
| Runtime variable | During execution | Values produced or supplied at runtime |
| Secret variable | During execution, masked | Sensitive values |

Syntax:

```yaml
parameters:
  - name: deployReports
    type: boolean
    default: true

variables:
  buildConfiguration: validation

steps:
  - pwsh: Write-Host "Mode is $(buildConfiguration)"

  - ${{ if eq(parameters.deployReports, true) }}:
      - pwsh: Write-Host "Reports are in scope"
```

Common expressions:

- `${{ }}`: compile-time template expression;
- `$[ ]`: runtime expression;
- `$(name)`: macro variable substitution.

Do not place secrets in compile-time parameters.

Do not worry if the expression syntax feels unfamiliar. First become comfortable with indentation, mappings, sequences, stages, jobs, and steps. Return to expressions after running the first pipeline.

## 3.12 Build and Release Separation

A build answers: **Is this change releasable?**  
A release answers: **Should this known release be deployed to this environment now?**

Separate pipelines can provide stronger isolation. A unified multi-stage pipeline is easier for a small team. In either model, publish a release artifact and deploy that artifact rather than rebuilding different content for each environment.

## Lab 3 - YAML Beginner Bootcamp

### Part A - Scalars

Create `yaml-practice.yml`:

```yaml
course: Fabric CI/CD
week: 3
completed: false
targetEnvironment: uat
```

Change each value and explain its type.

### Part B - Lists and mappings

Add:

```yaml
itemTypes:
  - Notebook
  - Lakehouse
  - DataPipeline

environment:
  name: uat
  requiresApproval: true
```

Point to one sequence, one mapping, two keys, and two scalar values.

### Part C - First runnable pipeline

Create this pipeline in a training Azure DevOps project:

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

steps:
  - script: echo "I can read YAML"
    displayName: First step

  - script: |
      echo "This is a second step"
      echo "It has two commands"
    displayName: Second step
```

Queue it manually and find both step names in the run.

### Part D - Find and fix errors

Fix every problem:

```yaml
trigger none

pool:
vmImage: ubuntu-latest

steps:
  script: echo "Validate"
    displayName: Validate
  - script echo "Package"
  displayName: Package
```

Expected result:

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

steps:
  - script: echo "Validate"
    displayName: Validate

  - script: echo "Package"
    displayName: Package
```

### Part E - Label an Azure Pipeline

Print or copy the Minimal Fabric CI Pipeline and label:

- trigger;
- parameters;
- variables;
- stages;
- jobs;
- steps;
- tasks;
- pool;
- artifact;
- environment; and
- approval boundary.

Then create a two-stage pipeline with `Validate` and `Package` stages. Make `Package` depend on `Validate`.

## Quiz 3

1. Is YAML a programming language?
2. What is a scalar?
3. What does a dash mean in a YAML sequence?
4. What does indentation mean in YAML?
5. What is the difference between a mapping and a sequence?
6. When should you use `|` for a script?
7. When should you use a parameter instead of a variable?
8. What does `dependsOn` control?
9. Why should a release deploy an artifact rather than re-read changing source?

---

# Module 4 - Fabric CI/CD Architecture

## Objectives

- Explain Fabric item definitions and logical identities.
- Compare Git integration, deployment pipelines, REST APIs, Fabric CLI, and `fabric-cicd`.
- Select an appropriate deployment route.
- Identify unsupported or partially supported item types.

## 4.1 The Fabric CI/CD Stack

Microsoft Fabric CI/CD combines:

1. **Git integration** for source control;
2. **Deployment pipelines** for staged promotion;
3. **Fabric REST APIs** as the automation foundation;
4. **Variable libraries** for environment configuration;
5. **Fabric CLI** for scriptable operations;
6. **Terraform** for infrastructure provisioning; and
7. **`fabric-cicd`** for source-controlled item deployment.

These are complementary, not mutually exclusive.

## 4.2 Item Definitions

A Fabric item definition is a folder containing the serialized representation of an item. It normally includes a `.platform` file with metadata such as:

- display name;
- item type; and
- `logicalId`.

Never casually edit a `logicalId`. Fabric uses it to track logical items across Git branches and workspaces.

## 4.3 Capability Questions for Every Item Type

Before automating an item, confirm:

| Question | Why it matters |
|---|---|
| Does Git integration support it? | Determines whether the definition can be synchronized |
| Can REST APIs create/update it from a definition? | Determines whether tools can deploy it |
| Can a service principal operate on it? | Determines unattended automation |
| Does it support deployment pipelines? | Determines native promotion support |
| Does it read variable libraries? | Determines configuration strategy |
| Does it auto-bind dependencies? | Determines whether post-deployment repair is needed |

Build a support matrix for your solution. Recheck it before major releases because Fabric capabilities change.

## 4.4 Choosing a Deployment Route

| Route | Best fit | Main caution |
|---|---|---|
| Fabric deployment pipelines | Native staged promotion and auto-binding | Item support and deployment rules vary |
| `fabric-cicd` | Git-first, Azure DevOps-driven item deployment | Requires Python automation and careful parameterization |
| Fabric REST APIs | Maximum control and custom orchestration | You own polling, errors, ordering, and mapping |
| Fabric CLI | Operator-friendly scripts and CI/CD | Verify command maturity and service principal support |
| Terraform | Workspace, capacity, and platform infrastructure | Not a replacement for every content deployment |

## 4.5 Recommended Reference Architecture

For this course:

- connect only the Development workspace to Azure Repos;
- use PRs as the review boundary;
- create one immutable release artifact from reviewed source;
- deploy definitions with `fabric-cicd`;
- use a Fabric variable library where supported;
- use parameter-file replacement only when required;
- protect UAT and Production with Azure DevOps environments;
- validate after every deployment;
- keep normal developers read-only in higher environments; and
- prevent direct Production edits.

## Lab 4 - Create a Capability Matrix

For each Sales Analytics item, record:

- Git support;
- deployment pipeline support;
- REST API support;
- service principal support;
- variable library support;
- auto-binding behavior;
- dependency order; and
- manual post-deployment action.

Do not continue until every unknown has an owner and a verification action.

## Quiz 4

1. Why is Fabric REST API support important even if you use a higher-level tool?
2. What does auto-binding protect you from?
3. Why should `logicalId` not be edited casually?
4. When is `fabric-cicd` a better fit than a native deployment pipeline?

---

# Module 5 - Fabric Git Integration and Team Development

## Objectives

- Connect a Development workspace to Azure Repos.
- Understand workspace and branch synchronization.
- Avoid shared-workspace conflicts.
- Resolve drift deliberately.

## 5.1 Basic Workflow

```text
Developer edits Development workspace
    -> Commit to connected branch
    -> Review item definition diff
    -> Pull request
    -> CI validation
    -> Merge
```

Fabric Git integration synchronizes definitions, not the complete runtime state of your solution. Data and many external resources remain outside the repository.

## 5.2 Shared Development Workspace Risks

A shared workspace has one live version of each item. Git branches do not automatically create isolated live workspace copies.

Risks:

- one developer overwrites another's item;
- a workspace contains uncommitted changes;
- the connected branch and workspace differ;
- portal edits happen while a PR is being reviewed;
- dependencies refer to another workspace.

For larger teams, evaluate branch-out or developer workspaces. Define ownership rules for shared items.

## 5.3 Drift Rules

Choose and enforce a policy:

- no direct edits in UAT or Production;
- emergency Production changes require an incident record;
- every emergency change must be back-ported to Git;
- deployment identities are separate from developer identities;
- workspace changes are compared before release; and
- a release pauses if the target has unauthorized drift.

## 5.4 Dependency Management

Deploy dependencies before dependants:

```text
Variable library / connections
    -> Lakehouse / Warehouse
    -> Notebook
    -> Data Pipeline
    -> Semantic Model
    -> Report
```

This is a starting order, not a universal law. Build the graph for your actual solution.

## Lab 5 - Synchronize and Review

1. Connect `SalesAnalytics-Dev` to an Azure Repos branch.
2. Commit the course project items.
3. Change one notebook and one pipeline.
4. Inspect the generated file changes.
5. Undo one workspace change from Git.
6. Document what Fabric did and did not synchronize.

## Quiz 5

1. Does a Git branch create an isolated Fabric workspace automatically?
2. Why are direct UAT edits dangerous?
3. What should happen after an emergency Production fix?
4. Why must dependency order be explicit?

---

# Module 6 - Environment Configuration and Secrets

## Objectives

- Separate code, configuration, and secrets.
- Use Fabric variable libraries and value sets.
- Use Azure Key Vault-linked variable groups.
- Avoid hardcoded GUIDs.

## 6.1 Three Categories

| Category | Examples | Storage |
|---|---|---|
| Definition | Notebook code, pipeline definition, report definition | Git |
| Non-secret configuration | Workspace name, server name, item reference | Variable library or controlled configuration |
| Secret | Client secret, password, token | Azure Key Vault |

Never store a secret in:

- YAML;
- a notebook;
- a parameter file;
- a variable library intended for non-secret settings;
- command output;
- a pipeline artifact; or
- source history.

## 6.2 Variable Library First

Use a Fabric variable library when the item type supports it.

Example notebook access:

```python
database_server = notebookutils.variableLibrary.get(
    "$(/**/environment_settings/database_server)"
)
database_name = notebookutils.variableLibrary.get(
    "$(/**/environment_settings/database_name)"
)
```

Create `dev`, `uat`, and `prod` value sets. Activate the correct value set in each workspace. The definition can remain identical while the workspace-level active value set differs.

Use:

- connection reference variables for external connections;
- item reference variables for dependencies; and
- typed values instead of stringly typed configuration where possible.

## 6.3 Azure DevOps Variable Groups

Create:

1. `fabric-cicd-sensitive`, linked to Azure Key Vault; and
2. `fabric-cicd-settings`, containing non-secret names and paths.

Example non-secret values:

```text
uatWorkspaceName=SalesAnalytics-UAT
prodWorkspaceName=SalesAnalytics-Prod
repositoryDirectory=fabric
```

Authorize only the required pipelines. Secret variable groups are protected resources and can have checks and pipeline permissions.

## 6.4 Parameter File Fallback

When an item cannot read a variable library, `fabric-cicd` can replace environment-specific values.

```yaml
find_replace:
  - find_value: "00000000-0000-0000-0000-000000000001"
    replace_value:
      uat: "$workspace.$id"
      prod: "$workspace.$id"

  - find_value: "00000000-0000-0000-0000-000000000002"
    replace_value:
      uat: "$items.Lakehouse.SalesLakehouse.$id"
      prod: "$items.Lakehouse.SalesLakehouse.$id"
```

Prefer symbolic references over maintaining separate target GUIDs. Scope replacements narrowly and test the rendered definition. A broad text replacement can modify unintended content.

## 6.5 Identity Design

For training, a service principal with a client secret is simple. For enterprise use:

- prefer workload identity federation where the supported toolchain allows it;
- use separate identities for non-Production and Production;
- grant least privilege;
- restrict who can use the Production service connection;
- rotate credentials;
- never print access tokens; and
- enable the required Fabric tenant settings only for an approved security group.

## Lab 6 - Remove Hardcoded Values

Find all environment-specific values in the course project:

- workspace IDs;
- lakehouse IDs;
- SQL endpoint IDs;
- connection IDs;
- storage paths;
- workspace names; and
- credentials.

Classify each as definition, configuration, or secret. Move supported configuration into `environment_settings.VariableLibrary`. Store secrets in Key Vault. Add parameter replacements only for remaining unsupported cases.

## Quiz 6

1. Why is a masked pipeline variable not a substitute for Key Vault governance?
2. What is the advantage of a variable library value set?
3. When should `find_replace` be a fallback?
4. Why should Production use a different identity from non-Production?

---

# Module 7 - Building a Pull Request CI Pipeline

## Objectives

- Validate changed Fabric definitions.
- Publish validation evidence.
- Fail clearly on invalid input.
- Make CI a required branch policy.

## 7.1 CI Quality Gates

Start with:

1. repository structure validation;
2. YAML and JSON parsing;
3. required `.platform` files;
4. duplicate logical ID detection;
5. forbidden secret and hardcoded-value detection;
6. item support matrix enforcement;
7. notebook linting and unit tests where practical;
8. dependency graph validation;
9. semantic model checks;
10. release manifest generation.

## 7.2 Example CI Pipeline

```yaml
name: fabric-ci-$(Date:yyyyMMdd).$(Rev:r)

trigger: none

pr:
  branches:
    include:
      - main
  paths:
    include:
      - fabric/**
      - pipelines/**
      - scripts/**
      - tests/**

pool:
  vmImage: ubuntu-latest

variables:
  pythonVersion: "3.12"

stages:
  - stage: Validate
    jobs:
      - job: StaticValidation
        steps:
          - checkout: self
            fetchDepth: 0

          - task: UsePythonVersion@0
            inputs:
              versionSpec: $(pythonVersion)

          - script: python -m pip install --require-hashes -r requirements-ci.txt
            displayName: Install pinned validation dependencies

          - script: python scripts/validate_fabric_items.py --root fabric
            displayName: Validate Fabric item definitions

          - script: python -m pytest tests/ci --junitxml=$(Common.TestResultsDirectory)/ci.xml
            displayName: Run CI tests

          - task: PublishTestResults@2
            condition: always()
            inputs:
              testResultsFormat: JUnit
              testResultsFiles: $(Common.TestResultsDirectory)/ci.xml
              failTaskOnFailedTests: true

  - stage: Package
    dependsOn: Validate
    jobs:
      - job: CreateReleaseArtifact
        steps:
          - checkout: self

          - task: CopyFiles@2
            inputs:
              sourceFolder: $(Build.SourcesDirectory)
              contents: |
                fabric/**
                scripts/deploy_fabric.py
                .deploy/parameter.yml
                requirements-deploy.txt
              targetFolder: $(Build.ArtifactStagingDirectory)/release

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: $(Build.ArtifactStagingDirectory)/release
              artifact: fabric-release
```

## 7.3 Validation Design Principles

- Every failure message must say what failed, where, and how to fix it.
- Do not convert validation errors into warnings merely to keep the pipeline green.
- Publish test results even when tests fail.
- Pin deployment dependencies and review upgrades.
- Generate a manifest containing commit, item names, item types, checksums, and tool versions.
- Never download unreviewed scripts during a release.

## Lab 7 - Break the Build

Create and detect these faults one at a time:

1. malformed JSON;
2. missing `.platform`;
3. duplicate `logicalId`;
4. hardcoded Development workspace ID;
5. unsupported item type;
6. a simulated secret string; and
7. invalid dependency reference.

For each failure, confirm the pipeline exits nonzero and provides an actionable message.

## Quiz 7

1. Why should CI use `trigger: none` when it is dedicated to PR validation?
2. Why publish test results with `condition: always()`?
3. What information belongs in a release manifest?
4. Why pin deployment dependencies?

---

# Module 8 - Deployment with `fabric-cicd`

## Objectives

- Authenticate unattended automation.
- Initialize a Fabric deployment workspace.
- Deploy selected item types.
- Handle destructive cleanup safely.

## 8.1 Prerequisites

- Dev, UAT, and Production workspaces exist.
- The automation identity can access the target workspaces.
- The required Fabric tenant setting permits approved service principals.
- Item types are supported.
- Key Vault and Azure DevOps variables are configured.
- UAT and Production Azure DevOps environments exist.
- The release artifact contains definitions, scripts, and locked dependencies.

## 8.2 Safe Deployment Script

The following teaching example uses workspace IDs supplied by protected configuration. It intentionally makes orphan deletion opt-in.

```python
import argparse
import ast
from pathlib import Path

from azure.identity import ClientSecretCredential
from fabric_cicd import (
    FabricWorkspace,
    publish_all_items,
    unpublish_all_orphan_items,
)


def parse_item_types(raw_value: str) -> list[str]:
    value = ast.literal_eval(raw_value)
    if not isinstance(value, list) or not all(
        isinstance(item, str) and item for item in value
    ):
        raise ValueError("items-in-scope must be a non-empty list of strings")
    return value


def main() -> None:
    parser = argparse.ArgumentParser()
    parser.add_argument("--tenant-id", required=True)
    parser.add_argument("--client-id", required=True)
    parser.add_argument("--client-secret", required=True)
    parser.add_argument("--workspace-id", required=True)
    parser.add_argument("--environment", choices=["uat", "prod"], required=True)
    parser.add_argument("--repository-directory", type=Path, required=True)
    parser.add_argument("--items-in-scope", required=True)
    parser.add_argument("--remove-orphans", action="store_true")
    args = parser.parse_args()

    if not args.repository_directory.is_dir():
        raise FileNotFoundError(
            f"Repository directory not found: {args.repository_directory}"
        )

    credential = ClientSecretCredential(
        tenant_id=args.tenant_id,
        client_id=args.client_id,
        client_secret=args.client_secret,
    )

    target = FabricWorkspace(
        workspace_id=args.workspace_id,
        environment=args.environment,
        repository_directory=str(args.repository_directory),
        item_type_in_scope=parse_item_types(args.items_in_scope),
        token_credential=credential,
    )

    publish_all_items(target)

    if args.remove_orphans:
        unpublish_all_orphan_items(target)


if __name__ == "__main__":
    main()
```

## 8.3 Why Orphan Deletion Is Dangerous

`unpublish_all_orphan_items()` can delete target items of an in-scope type when they are absent from the source branch.

Enable it only when:

- the release artifact represents the complete desired state;
- all target-owned exceptions are explicitly excluded;
- the release plan displays the deletions;
- UAT proves the behavior; and
- Production deletion requires appropriate approval.

Do not combine selective deployment with automatic orphan deletion unless you fully understand the resulting desired-state scope.

## 8.4 Deployment YAML Template

```yaml
# pipelines/templates/deploy-fabric.yml
parameters:
  - name: stageName
    type: string
  - name: displayName
    type: string
  - name: environmentName
    type: string
  - name: workspaceIdVariable
    type: string
  - name: dependsOn
    type: object
    default: []

stages:
  - stage: ${{ parameters.stageName }}
    displayName: ${{ parameters.displayName }}
    dependsOn: ${{ parameters.dependsOn }}
    jobs:
      - deployment: DeployFabric
        environment: ${{ parameters.environmentName }}
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: fabric-release

                - task: UsePythonVersion@0
                  inputs:
                    versionSpec: "3.12"

                - script: >-
                    python -m pip install
                    --require-hashes
                    -r "$(Pipeline.Workspace)/fabric-release/requirements-deploy.txt"
                  displayName: Install pinned deployment dependencies

                - task: PythonScript@0
                  displayName: Deploy Fabric items
                  inputs:
                    scriptSource: filePath
                    scriptPath: $(Pipeline.Workspace)/fabric-release/scripts/deploy_fabric.py
                    arguments: >-
                      --tenant-id "$(fabricTenantId)"
                      --client-id "$(fabricClientId)"
                      --client-secret "$(fabricClientSecret)"
                      --workspace-id "$(${{ parameters.workspaceIdVariable }})"
                      --environment "${{ parameters.environmentName }}"
                      --repository-directory "$(Pipeline.Workspace)/fabric-release/fabric"
                      --items-in-scope
                      "['VariableLibrary','Lakehouse','Notebook','DataPipeline','SemanticModel','Report']"
```

Secrets are passed as task arguments here for readability. Prefer mapping secrets through task environment variables when the deployment script supports that interface, because command lines can be exposed in process diagnostics.

## Lab 8 - Deploy to UAT

1. Create the non-Production deployment identity.
2. Add it to the UAT workspace with the minimum working role.
3. Configure the Key Vault-linked variable group.
4. Deploy the release artifact to UAT.
5. Verify the item count and definitions.
6. Confirm Development IDs do not remain in UAT.
7. Rerun the same release and check idempotency.
8. Remove one harmless test item from source and preview cleanup behavior without enabling deletion.

## Quiz 8

1. Why is idempotency important?
2. Why does the example require an explicit `--remove-orphans` switch?
3. Why deploy a pipeline artifact rather than checking out `main` again?
4. What is the risk of passing a secret on a command line?

---

# Module 9 - Multi-Stage CD, Approvals, and Production Protection

## Objectives

- Create UAT and Production deployment stages.
- Configure approvals outside YAML.
- Use branch control and exclusive locking.
- Promote the same artifact.

## 9.1 Multi-Stage Pipeline

```yaml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - fabric/**
      - pipelines/**
      - scripts/**

variables:
  - group: fabric-cicd-sensitive
  - group: fabric-cicd-settings

stages:
  - stage: Validate
    jobs:
      - job: Validate
        steps:
          - checkout: self
          - script: python scripts/validate_fabric_items.py --root fabric

  - stage: Package
    dependsOn: Validate
    jobs:
      - job: Package
        steps:
          - checkout: self
          - task: CopyFiles@2
            inputs:
              sourceFolder: $(Build.SourcesDirectory)
              contents: |
                fabric/**
                scripts/deploy_fabric.py
                .deploy/parameter.yml
                requirements-deploy.txt
              targetFolder: $(Build.ArtifactStagingDirectory)/release
          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: $(Build.ArtifactStagingDirectory)/release
              artifact: fabric-release

  - template: templates/deploy-fabric.yml
    parameters:
      stageName: Deploy_UAT
      displayName: Deploy to UAT
      environmentName: uat
      workspaceIdVariable: uatWorkspaceId
      dependsOn:
        - Package

  - stage: Validate_UAT
    dependsOn: Deploy_UAT
    jobs:
      - job: SmokeTests
        steps:
          - script: python tests/post_deploy.py --environment uat

  - template: templates/deploy-fabric.yml
    parameters:
      stageName: Deploy_Production
      displayName: Deploy to Production
      environmentName: prod
      workspaceIdVariable: prodWorkspaceId
      dependsOn:
        - Validate_UAT
```

## 9.2 Checks Are Not Controlled by YAML Authors

Configure Azure DevOps approvals and checks on resources such as:

- environments;
- service connections;
- repositories;
- variable groups;
- secure files; and
- agent pools.

This separation prevents a pipeline author from removing the Production approval by editing YAML.

## 9.3 Recommended Production Checks

| Check | Purpose |
|---|---|
| Branch control | Allow only protected `refs/heads/main` |
| Required template | Enforce the enterprise deployment template |
| Pre-approval | Require release owner or business approver |
| Business hours | Restrict deployment window |
| Invoke REST API / Function | Evaluate automated quality or change-management policy |
| Azure Monitor query | Block deployment during unhealthy conditions |
| Exclusive lock | Prevent concurrent Production releases |
| Post-approval | Support final verification where policy requires it |

Service connections cannot be selected dynamically through a variable. Design templates accordingly.

## 9.4 Approval Evidence

An approver should receive:

- release version and commit;
- changed items;
- additions, updates, and deletions;
- CI results;
- UAT results;
- configuration changes;
- dependency risks;
- security review result;
- rollback version; and
- release owner.

"Pipeline is green" is not enough information for a responsible approval.

## Lab 9 - Protect Production

Configure:

1. UAT pre-approval;
2. Production branch control;
3. Production pre-approval;
4. exclusive lock;
5. a timeout;
6. restricted pipeline access to the Production secret group; and
7. a failed smoke test that prevents Production.

Verify that editing YAML cannot bypass the environment approval.

## Quiz 9

1. Why are Azure DevOps environment checks configured outside YAML?
2. What problem does exclusive lock solve?
3. Why must branch control use a fully qualified branch name?
4. What evidence should an approver inspect?

---

# Module 10 - Fabric REST API and Native Deployment Pipeline Automation

## Objectives

- Understand when direct API automation is appropriate.
- Handle long-running operations.
- Selectively deploy items.
- Compare API promotion with definition-based deployment.

## 10.1 Important Deployment Pipeline APIs

Fabric APIs can:

- create and update a deployment pipeline;
- list pipeline stages;
- assign workspaces to stages;
- list stage items;
- deploy all or selected stage content;
- inspect operations; and
- retrieve long-running operation results.

## 10.2 Long-Running Operation Pattern

```text
POST deploy
    -> receive operation reference
    -> poll operation state
    -> stop on success or terminal failure
    -> retrieve result
    -> validate target
```

Production-quality polling must include:

- timeout;
- retry interval or backoff;
- handling for throttling;
- terminal failure detection;
- operation ID logging;
- response body capture without secrets; and
- a nonzero exit code on failure.

## 10.3 Selective Deployment

Selective deployment is useful when:

- a release contains independently deployable items;
- a critical fix must minimize scope; or
- large solutions require coordinated waves.

It is dangerous when:

- dependencies are omitted;
- the source stage no longer matches reviewed content;
- the selected item binds to a newer unreviewed shared item; or
- the operator assumes PR approval isolates only one merged change.

Always calculate and present dependency closure.

## 10.4 Choosing Between Two CD Patterns

### Pattern A - Deploy definitions with `fabric-cicd`

```text
Reviewed Git artifact -> UAT workspace -> Production workspace
```

Advantages:

- Git content is directly deployable;
- release artifact can be immutable;
- flexible parameter replacement;
- strong Azure DevOps integration.

### Pattern B - Automate native Fabric deployment pipelines

```text
Reviewed Development stage -> UAT stage -> Production stage
```

Advantages:

- native stage comparison;
- native content promotion;
- native relationship handling for supported items;
- operator familiarity.

Critical control: if the API promotes from a live source workspace or stage, verify it still matches the reviewed commit or package. A later direct edit can otherwise enter the release.

## Lab 10 - Design an API Client

Create pseudocode or a non-Production script that:

1. obtains an Entra token for Fabric;
2. resolves the deployment pipeline and stages;
3. lists source items;
4. calculates selected items and dependencies;
5. starts deployment;
6. polls to completion;
7. records the operation result; and
8. invokes post-deployment validation.

Test timeout and terminal failure paths.

## Quiz 10

1. Why is a `202 Accepted` response not proof of deployment success?
2. What is dependency closure?
3. What drift risk exists when promoting from a live source stage?
4. When is native deployment pipeline automation preferable?

---

# Module 11 - Testing, Validation, and Observability

## Objectives

- Build layered release validation.
- Define measurable gates.
- Produce audit evidence.
- Monitor deployment and runtime health.

## 11.1 Test Pyramid for Fabric

### Static tests

- definition schema;
- JSON/YAML syntax;
- naming standards;
- forbidden hardcoded values;
- duplicate identities;
- unsupported item types;
- dependency graph.

### Unit tests

- pure notebook functions;
- transformation logic;
- reusable Python modules;
- data quality rules;
- configuration parsing.

### Integration tests

- connection access;
- lakehouse table availability;
- notebook execution;
- pipeline execution;
- semantic model refresh;
- report-to-model binding.

### Data tests

- schema;
- row counts;
- null thresholds;
- uniqueness;
- referential integrity;
- metric reconciliation;
- freshness.

### Release tests

- expected items deployed;
- environment references correct;
- no Development IDs in target;
- schedules deliberately enabled or disabled;
- ownership and permissions correct;
- monitored job completes.

## 11.2 PASS/WARN/FAIL Contract

Every gate should define:

| Result | Meaning | Pipeline action |
|---|---|---|
| PASS | Requirement met | Continue |
| WARN | Accepted risk within policy | Continue and record |
| FAIL | Requirement not met | Stop |

Avoid ambiguous tests such as "row count looks reasonable." Use measurable thresholds:

```text
FAIL if source-to-target row difference exceeds 0.5%
FAIL if required column is absent
FAIL if freshness exceeds 90 minutes
WARN if query duration increases by 20-30%
FAIL if query duration increases by more than 30%
```

Thresholds must come from business and technical requirements, not arbitrary defaults.

## 11.3 Release Evidence Bundle

Publish:

- manifest;
- checksums;
- tool versions;
- test results;
- environment;
- operation IDs;
- item-level deployment result;
- validation queries and outputs;
- approver identity;
- timestamps;
- rollback release.

Retain evidence according to audit policy and avoid including sensitive data.

## 11.4 Monitoring

Monitor both:

1. **deployment health** - did the release complete correctly?
2. **solution health** - does the deployed solution operate correctly?

Track:

- deployment success rate;
- deployment duration;
- change failure rate;
- mean time to restore;
- failed notebook and pipeline runs;
- semantic model refresh failures;
- freshness and data quality;
- capacity pressure;
- unauthorized drift.

## Lab 11 - Build a Release Test Pack

Implement at least:

1. item inventory comparison;
2. forbidden Development GUID scan;
3. required table schema check;
4. row-count reconciliation;
5. notebook or data pipeline smoke run;
6. semantic model refresh check;
7. release evidence summary.

Force each test to fail once and confirm the pipeline blocks the release.

## Quiz 11

1. Why is successful deployment not sufficient verification?
2. What makes a quality gate measurable?
3. What is change failure rate?
4. Why should validation output avoid sensitive data?

---

# Module 12 - Rollback, Troubleshooting, Security, and Scale

## Objectives

- Design rollback before release.
- Diagnose failures systematically.
- Harden the supply chain.
- scale from one pipeline to a platform.

## 12.1 Rollback Is a Release

Rollback should mean redeploying a known good immutable release, not manually editing Production.

Before deployment, record:

- current Production release;
- target release;
- data/schema compatibility;
- reversible and irreversible steps;
- rollback owner;
- maximum restore time.

Definitions can often be rolled back. Data mutations might not be reversible. Separate definition rollback from data recovery.

## 12.2 Roll Forward versus Roll Back

| Situation | Preferred response |
|---|---|
| Small definition defect, no data impact | Roll back or rapid roll forward |
| Irreversible schema/data change | Controlled roll forward |
| Security exposure | Disable access, contain, then remediate |
| Configuration-only defect | Restore last known configuration |
| Partial deployment | Stop, assess target consistency, redeploy complete known release |

## 12.3 Troubleshooting Sequence

1. Identify the first failing stage and step.
2. Read the complete error, status code, and correlation/operation ID.
3. Confirm which identity executed the action.
4. Confirm tenant, workspace, environment, branch, and artifact version.
5. Check permissions and Fabric tenant settings.
6. Verify item-type capability and API support.
7. Check parameter rendering and remaining source IDs.
8. Inspect dependencies and deployment order.
9. Check service health and throttling.
10. Reproduce in a safe environment with the same artifact.

Do not "fix" an authorization error by granting broad administrator access without determining the required permission.

## 12.4 Common Failure Matrix

| Symptom | Likely cause | Investigation |
|---|---|---|
| Workspace not found | Wrong identity, name, or ID | List identity-visible workspaces |
| 401 | Invalid/expired credential or token audience | Inspect auth configuration |
| 403 | Missing role, tenant setting, or item support | Check least-privilege path |
| 409 | Conflict, duplicate identity, concurrent operation | Inspect logical IDs and locks |
| 429 | Throttling | Honor retry guidance and back off |
| Item points to Dev | Missing parameterization or binding | Scan deployed definition |
| Pipeline never starts | Trigger/path/branch policy mismatch | Inspect run and policy configuration |
| Deployment waits forever | Approval/check/lock pending | Inspect environment checks |
| Report broken after deploy | Semantic model dependency not rebound | Validate dependency graph |
| Unexpected deletion | Orphan cleanup scope incorrect | Stop release and restore known version |

## 12.5 Security Hardening

- use least privilege;
- separate Production identity;
- prefer short-lived federated credentials when supported;
- store secrets in Key Vault;
- restrict service connection and variable group use;
- require protected templates;
- pin dependencies;
- scan source and artifacts for secrets;
- restrict self-hosted agent network access;
- prevent pull requests from untrusted forks from receiving secrets;
- review third-party tasks;
- retain audit logs;
- prohibit direct Production edits;
- include emergency access procedures.

## 12.6 Scaling to a Platform

Move repeated logic into centrally governed templates:

```text
platform-pipelines/
|-- templates/
|   |-- fabric-ci.yml
|   |-- fabric-deploy.yml
|   |-- fabric-validate.yml
|   `-- fabric-release-summary.yml
|-- policies/
|-- schemas/
`-- version.txt
```

Product repositories provide:

- item definitions;
- dependency manifest;
- environment metadata references;
- test specifications;
- approved template version.

The platform team owns:

- deployment templates;
- identity standards;
- policy checks;
- common validation;
- observability;
- upgrade testing.

Version templates. A breaking template change must not silently affect every Fabric solution.

## Lab 12 - Game Day

Run a controlled failure exercise:

1. deploy a known good release to UAT;
2. introduce a bad environment reference;
3. confirm validation detects it;
4. simulate a partial deployment;
5. execute the rollback runbook;
6. calculate time to restore;
7. document evidence gaps;
8. improve the pipeline.

## Quiz 12

1. Why can item-definition rollback fail to restore data?
2. What should you investigate before increasing permissions?
3. Why must shared templates be versioned?
4. What is the first action after detecting an unexpected deletion?

---

# Capstone - Production-Grade Fabric Release System

## Scenario

CMCO is migrating a Sales Analytics solution to Fabric. The organization requires:

- Azure DevOps YAML as the PR-controlled route;
- environment-specific parameterization;
- protected approvals;
- protected deployment identities;
- schema, data quality, and dependency validation;
- UAT evidence before Production;
- no ordinary developer write access in Production;
- monitoring and operational runbooks; and
- rollback by redeploying a previous known release.

## Required Deliverables

1. Architecture diagram.
2. Repository structure.
3. Item capability matrix.
4. Dependency graph.
5. Branch and PR policy.
6. CI YAML.
7. reusable deployment template.
8. UAT and Production pipeline.
9. Key Vault and identity design.
10. variable library/value-set design.
11. parameter replacement file for unsupported cases.
12. validation test pack.
13. release evidence report.
14. rollback runbook.
15. security threat review.
16. operations and ownership matrix.

## Acceptance Criteria

### Source control

- All deployable definitions are versioned.
- Production cannot be updated from an unprotected branch.
- The release maps to an immutable commit and artifact.

### Security

- No secret exists in Git or artifacts.
- UAT and Production identities are appropriately separated.
- Production approvals cannot be removed through a YAML edit.
- The Production identity has only required permissions.

### Deployment

- The same artifact reaches UAT and Production.
- Environment values resolve correctly.
- Dependencies deploy in a verified order.
- Rerunning the same release is safe.
- Deletions are visible and controlled.

### Validation

- Invalid definitions fail CI.
- UAT smoke, schema, quality, and dependency tests run.
- Production is blocked when required UAT checks fail.
- Post-deployment verification produces evidence.

### Recovery

- A known-good version can be redeployed.
- Irreversible data changes are identified.
- The team completes a rollback game day.

## Capstone Scoring Rubric

| Area | Weight |
|---|---:|
| Architecture and item support analysis | 15% |
| Git and CI design | 15% |
| YAML quality and reuse | 15% |
| Secure identity and secrets | 15% |
| Deployment correctness | 15% |
| Validation and observability | 15% |
| Rollback and operations | 10% |

**Mastery target:** 85% or higher, with no critical security or recovery gap.

---

# Quiz Answer Key

## Module 1

1. Integrate changes frequently and validate the combined codebase early.
2. It retains a human authorization point before a high-impact release.
3. The reviewed release content; only approved environment configuration should differ.
4. A stage is a major workflow boundary; a step is an action inside a job.

## Module 2

1. A workspace is live mutable state and does not provide the complete reviewed version history.
2. It reduces divergence and makes changes easier to review and integrate.
3. To prevent unreviewed or policy-bypassing content from reaching Production.
4. A tag identifies an immutable version, while a branch can move.

## Module 3

1. No. YAML describes structured data and configuration; tasks and scripts perform the work.
2. One value, such as a string, number, or Boolean.
3. It starts an item in an ordered list.
4. It defines parent-child structure and shows which properties belong together.
5. A mapping contains named `key: value` properties; a sequence is an ordered list.
6. Use `|` when a script contains multiple separate commands and line breaks must be preserved.
7. Use a parameter for typed, queue-time, or template-structural choices.
8. It controls the execution dependency between stages or jobs.
9. Re-reading source can deploy content different from what was validated.

## Module 4

1. Higher-level Fabric automation tools ultimately depend on API capabilities.
2. Dependencies remaining bound to items in the source or wrong workspace.
3. Fabric uses it to track logical item identity.
4. When Git definitions are the release artifact and flexible item deployment/parameterization is required.

## Module 5

1. No.
2. They create drift from the reviewed source and make the release non-repeatable.
3. Back-port it to Git and restore Git as the source of truth.
4. Dependants can fail or bind incorrectly if dependencies do not exist.

## Module 6

1. Key Vault adds centralized access control, rotation, and audit; masking only reduces log exposure.
2. Identical definitions can use environment-specific runtime values.
3. When supported variable-library or binding mechanisms cannot solve the requirement.
4. To limit blast radius and independently control Production access.

## Module 7

1. The pipeline should run as a branch-policy validation rather than after every branch commit.
2. Test evidence should be available even when a test fails.
3. Commit, items, checksums, tool versions, and release scope.
4. To make builds reproducible and prevent an unreviewed package update.

## Module 8

1. A retry must converge safely instead of duplicating or corrupting state.
2. Cleanup can delete target items, so destructive behavior must be deliberate.
3. The artifact is the exact content that passed validation.
4. Process metadata or diagnostics can expose it.

## Module 9

1. Resource owners retain control even if a YAML author changes pipeline code.
2. Concurrent releases modifying the same environment.
3. Azure DevOps checks expect a name such as `refs/heads/main`.
4. Scope, evidence, risks, configuration, tests, and rollback version.

## Module 10

1. It usually means the asynchronous operation started, not that it completed.
2. The selected items plus every dependency required for a valid release.
3. The source can change after review.
4. When native staged comparison, promotion, and supported auto-binding are priorities.

## Module 11

1. Definitions can deploy successfully while bindings, runtime execution, data, or refresh fail.
2. It has an explicit rule, threshold, and expected action.
3. The percentage of deployments causing degraded service or requiring remediation.
4. Evidence stores often have broad retention and readership.

## Module 12

1. Definition deployment does not reverse data writes or destructive schema changes.
2. The executing identity, required permission, tenant settings, and item capability.
3. A central change can otherwise break all consumers without controlled adoption.
4. Stop further deployment, preserve evidence, assess impact, and begin controlled recovery.

---

# Mastery Checklist

You are ready to lead a Fabric CI/CD implementation when you can complete all of these without a tutorial:

- [ ] Explain the entire commit-to-Production flow.
- [ ] Write a multi-stage Azure Pipeline.
- [ ] Explain compile-time and runtime YAML expressions.
- [ ] Configure PR build validation.
- [ ] Build an item-type support matrix.
- [ ] Explain item definitions and logical IDs.
- [ ] Choose between deployment pipelines, REST APIs, Fabric CLI, and `fabric-cicd`.
- [ ] Design variable library value sets.
- [ ] Eliminate hardcoded environment IDs.
- [ ] Authenticate an unattended deployment safely.
- [ ] Configure external approvals and branch control.
- [ ] Publish and promote one immutable artifact.
- [ ] Test item inventory, bindings, schemas, data, and refresh.
- [ ] Handle long-running Fabric API operations.
- [ ] Prevent unsafe orphan deletion.
- [ ] Diagnose 401, 403, 409, and 429 failures.
- [ ] Produce release evidence.
- [ ] Redeploy a known-good release.
- [ ] Explain data recovery separately from definition rollback.
- [ ] Scale the design with versioned templates.

---

# Suggested 13-Week Schedule

| Week | Study | Practical milestone |
|---|---|---|
| 1 | Module 1 | CI/CD vocabulary and manual-process map |
| 2 | Module 2 | Repository and branch policies |
| 3 | Module 3, sections 3.1-3.8 | YAML beginner exercises and first pipeline |
| 4 | Module 3, sections 3.9-3.12 | Working Fabric CI YAML |
| 5 | Module 4 | Architecture and capability matrix |
| 6 | Module 5 | Development workspace Git integration |
| 7 | Module 6 | Configuration and secret separation |
| 8 | Module 7 | Required PR validation |
| 9 | Module 8 | First UAT deployment |
| 10 | Module 9 | Protected Production stage |
| 11 | Modules 10-11 | API automation, test pack, and evidence |
| 12 | Module 12 | Rollback game day |
| 13 | Capstone | Demonstration and design defense |

---

# Official References

Use current Microsoft documentation as the source of truth:

- [What is CI/CD in Microsoft Fabric?](https://learn.microsoft.com/fabric/cicd/cicd-overview)
- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd)
- [Fabric Git integration](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration)
- [Fabric deployment pipelines](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/intro-to-deployment-pipelines)
- [Automate a Fabric deployment pipeline with APIs](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/pipeline-automation-fabric)
- [CI/CD for Fabric using Azure DevOps and `fabric-cicd`](https://learn.microsoft.com/fabric/cicd/tutorial-fabric-cicd-azure-devops)
- [Fabric CI/CD troubleshooting](https://learn.microsoft.com/fabric/cicd/troubleshoot-cicd)
- [Azure Pipelines YAML schema](https://learn.microsoft.com/azure/devops/pipelines/yaml-schema/)
- [Azure Pipelines templates](https://learn.microsoft.com/azure/devops/pipelines/process/templates)
- [Azure Pipelines environments](https://learn.microsoft.com/azure/devops/pipelines/process/environments)
- [Azure Pipelines approvals and checks](https://learn.microsoft.com/azure/devops/pipelines/process/approvals)
- [Azure Pipelines variable groups](https://learn.microsoft.com/azure/devops/pipelines/library/variable-groups)
- [Link variable groups to Azure Key Vault](https://learn.microsoft.com/azure/devops/pipelines/library/link-variable-groups-to-key-vaults)
- [`fabric-cicd` documentation](https://microsoft.github.io/fabric-cicd/)

---

## Final Principle

Mastery is not the ability to produce a large YAML file. It is the ability to prove that the correct reviewed Fabric release, using the correct protected identity and configuration, reached the correct environment, passed measurable validation, produced auditable evidence, and can be restored safely.
