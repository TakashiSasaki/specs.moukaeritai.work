# Universal Norms for Coding Agents

- Status: Draft
- Version: 0.1.0
- Audience: Coding agents that modify software repositories
- Scope: Projects under `moukaeritai.work` and comparable software-development environments

## 1. Purpose

These norms are intended to prevent regressions, over-constraint, responsibility drift, and uncontrolled scope expansion when a coding agent modifies specifications, implementation, tests, and verification infrastructure.

A coding agent must not assume that existing code is always correct. It must also not assume that validators or tests have the authority to define the specification they are meant to verify.

The objective is not to make every change infallible. The objective is to keep mistakes bounded, detectable, and recoverable through a small follow-up change.

## 2. Normative Terms

The following terms are used in this document:

- **MUST**: a requirement that is mandatory unless a higher-priority instruction explicitly overrides it.
- **MUST NOT**: an action that is prohibited.
- **SHOULD**: a recommended practice that should be followed unless there is a documented reason not to do so.
- **MAY**: an optional practice.

## 3. Classification Before Work Begins

Before beginning a significant change, the coding agent MUST identify the classification axes that materially affect the task.

### 3.1 Axes that must normally be identified

```text
Artifact type:
  specs / docs / code / tests / fixtures / scripts / config / generated

Implementation responsibility:
  UI / boundary / application / domain / infrastructure / verification /
  authentication / authorization / logging / monitoring

Normative role:
  authoritative / implementation / verification / explanation / example / derived

Change intent:
  preserve / correct / extend / remove / migrate / refactor / verify
```

### 3.2 Axes that must be identified when relevant

```text
Application interface surface:
  public / app / admin / dev / api / test

Execution environment or operational stage:
  production / staging / development / test / CI / build / release
```

The agent does not need to add this metadata to every file. It MUST, however, make the relevant classifications explicit in its work plan, change description, or final report whenever ambiguity could affect correctness.

## 4. Normative Dependency Direction

The coding agent MUST preserve the following dependency direction when determining correctness:

```text
authoritative specification / policy / contract
        ↓
implementation
        ↓
validator / tests / verification
```

### 4.1 Mandatory rules

1. Specifications, policies, contracts, and explicit human instructions take precedence over current implementation behavior.
2. Test expectations MUST be derived from an authoritative source or from an explicitly stated change intent.
3. Validators MUST verify rules that have already been defined.
4. The current implementation alone MUST NOT be used as the basis for inventing a new mandatory rule.

### 4.2 Prohibited inversion

The following dependency inversion is prohibited:

```text
current implementation
    ↓
test expectations copied from that behavior
    ↓
tests used to justify the implementation
```

If no authoritative basis for a constraint can be identified, the agent SHOULD report the issue instead of enforcing the constraint.

## 5. Do Not Infer Constraints Across Classification Axes

A coding agent MUST NOT automatically derive a constraint in one classification axis from a value in another classification axis.

The following inferences are specifically prohibited unless an authoritative source explicitly defines them:

```text
interface surface
  → mandatory URL prefix

interface surface
  → internal implementation layer

interface surface
  → execution environment

artifact type
  → normative authority

test artifact
  → source of truth

validator responsibility
  → authority to define new specifications

currently existing values
  → closed allowlist of all valid future values

preferred convention
  → mandatory invariant
```

For example, `surface = app` does not by itself imply that the path must begin with `/app`.

## 6. State the Change Intent Explicitly

For every non-trivial change, the coding agent MUST distinguish the following categories as applicable:

```text
Preserve:
  behavior that must remain unchanged

Correct:
  existing behavior that is known to be wrong and must change

Remove:
  identifiers, compatibility paths, branches, files, or behaviors to delete

Extend:
  behavior or accepted values to add

Out of scope:
  areas that must not be changed in this task
```

### 6.1 Refactor versus correction

- A `refactor` SHOULD preserve externally observable behavior.
- A `correct` change intentionally modifies existing incorrect behavior.

If both occur in one task, preserved behavior and corrected behavior MUST be listed separately.

A coding agent MUST NOT move an existing defect into a shared module merely because the task includes extraction or refactoring.

## 7. Keep Implementation Responsibilities Narrow

A module MUST NOT accumulate unrelated responsibilities without explicit justification.

A typical separation is:

```text
catalog validator:
  data structure
  required properties
  types
  enumerations
  documented invariants

runtime boundary checker:
  runtime registration
  guards
  wildcard coverage
  runtime boundaries

classifier:
  changed-file classification
  verification-plan construction

runner:
  Git operations
  subprocess execution
  exit-code handling

version checker:
  version synchronization
  version transition rules
```

Validators, classifiers, and policy checkers SHOULD be implemented as pure functions whenever practical.

A pure validation or classification function SHOULD NOT directly:

- read files;
- execute Git;
- launch subprocesses;
- terminate the process;
- interpret environment variables.

Those operations SHOULD be performed by an outer runner.

## 8. Testing Norms

### 8.1 Preserve concrete regressions

A regression that has occurred in practice MUST be represented by a concrete regression test when feasible.

The test SHOULD reproduce the failed input or condition instead of only using an abstract test name.

### 8.2 Pair positive and negative specifications

For each important constraint, the coding agent SHOULD provide both:

```text
Negative specification:
  a state that must be rejected

Positive specification:
  a valid state that must not be rejected
```

Negative tests alone do not protect against over-constraint.

### 8.3 Tests must use production logic

Tests MUST import and exercise production validation, classification, or transformation logic directly.

Prohibited pattern:

```text
production:
  classification regular expression

test:
  copied classification regular expression
```

Required pattern:

```text
production:
  classifyChangedFiles()

test:
  import classifyChangedFiles()
```

Tests MUST NOT reimplement production rules merely to confirm the same duplicated logic.

### 8.4 Test actual production data when practical

When feasible, tests SHOULD validate the actual production catalog, configuration, schema, or metadata using the same production validator used for fixtures.

## 9. Constraints on Verification Code

Verification code MUST NOT introduce undocumented constraints merely because they appear safer or match the current implementation.

The following are prohibited unless backed by an authoritative source:

- a closed allowlist containing only currently existing values;
- undocumented path-prefix requirements;
- a general prohibition on dynamic identifiers or dynamic paths;
- a new role taxonomy derived from existing access values;
- a list that rejects otherwise valid future extensions;
- treating an implementation accident as a permanent invariant.

A new verification condition MUST be supported by at least one of the following:

1. an authoritative source;
2. explicit human approval;
3. a concrete previous defect or regression;
4. a clear security, data-integrity, or external-contract requirement.

## 10. Treat Verification Infrastructure Conservatively

The following artifacts MUST be treated as verification infrastructure rather than ordinary implementation files:

```text
changed-file classifier
verification-plan builder
validator
version checker
bootstrap checker
boundary checker
verification runner
related tests
root manifest
root lockfile
CI workflow
```

When verification infrastructure changes, the selected verification plan MUST include a conservative baseline rather than relying only on ordinary changed-file classification.

The baseline SHOULD include at least:

```text
bootstrap validation
version synchronization
major unit tests
major boundary checks
type checking
build
```

If an entirely independent fixed gate is impractical, the conservative-plan logic MAY remain in a production classifier, provided that its behavior is protected by regression tests that call the production implementation directly.

## 11. Explicit Inputs Must Fail Closed

Inputs explicitly supplied by a user, CI system, or higher-level process MUST be distinguished from implicit defaults.

Examples include:

```text
Git base reference
branch
target environment
deployment target
configuration path
schema version
API endpoint
workspace package
```

If an explicit input is invalid, the operation MUST fail closed.

The following behavior is prohibited:

```text
explicit base reference is invalid
    ↓
silently fall back to HEAD~1
```

Fallback MAY be used only for implicit inputs and only in a documented order.

## 12. Control the Scope of Each Task

A task SHOULD have one primary objective.

If multiple objectives are unavoidable, the task SHOULD modify no more than one or two major classification axes at a time.

Before implementation, the coding agent MUST identify:

```text
Allowed files
Conditionally allowed files
Forbidden files
Out-of-scope findings
Required verification commands
```

Unrelated defects discovered during the task MUST NOT be added to the change without explicit approval. They SHOULD be reported or recorded as backlog candidates.

Before completion, the agent MUST review the changed-file list and revert unrelated modifications.

## 13. Use Observable Completion Criteria

Completion criteria MUST be observable and testable.

Insufficient criteria include:

```text
improve the design
make the system safer
add enough tests
```

Preferred criteria include:

```text
a named identifier no longer exists
a specific valid input succeeds
a specific invalid input fails
a test imports the production module
a required command exits with code 0
all changed files are inside the allowed scope
```

The coding agent MUST translate the change intent into concrete identifiers, inputs, outputs, errors, test results, or exit codes.

## 14. Execution and Reporting

For executed verification, the coding agent SHOULD report at least:

```text
command
exit code
test-file count
test-case count
material failure information
verification not executed
environment limitations
```

A command that was not executed MUST NOT be reported as successful.

If test-first work was required, the final report MUST distinguish:

```text
tests confirmed failing before the fix
tests confirmed passing after the fix
```

The agent MUST NOT claim test-first execution if it did not observe the pre-fix failure.

## 15. Priority When Full Compliance Is Impractical

If full compliance with these norms is impractical, the coding agent MUST prioritize the following in order:

1. identify the authoritative source;
2. state the change intent;
3. convert previous regressions into regression tests;
4. provide both positive and negative tests;
5. make tests call production logic directly;
6. avoid undocumented constraints;
7. keep the task scope narrow;
8. use conservative verification for verification-infrastructure changes;
9. fail closed on invalid explicit inputs;
10. connect important verification to the final gate.

Scope manifests, `CODEOWNERS`, fully independent fixed gates, and persistent test-first logs are useful, but they do not replace the core requirements above.

## 16. Final Review Checklist

Before completing a change, the coding agent MUST consider the following questions:

```text
- Is the source of truth clear?
- Are specification, implementation, and verification roles distinct?
- Has a constraint been inferred from an unrelated classification axis?
- Are Preserve, Correct, Remove, and Extend distinguished?
- Is a validator inventing a new specification?
- Is there a positive regression test?
- Is there a negative regression test?
- Do tests call production logic directly?
- Do verification-infrastructure changes select sufficient verification?
- Do invalid explicit inputs fail closed?
- Are all changed files inside the task scope?
- Is any unexecuted verification being presented as successful?
```

## 17. Core Directives

A coding agent MUST always follow these directives:

> Do not connect different classification axes in order to invent a constraint that has not been specified.

> Distinguish the artifacts that define the specification, implement it, and verify conformance to it. Do not reverse their dependency direction.

> Do not preserve the existing implementation unconditionally. Separate behavior that must be preserved from behavior that must be corrected.

> Preserve concrete regressions as tests, and protect not only states that must be rejected but also valid states that must remain accepted.

> Prefer a small number of effective and maintainable regression-prevention mechanisms over a large governance system that cannot be consistently followed.

## 18. Change History

- `0.1.0`: Initial English version. Defines classification-based universal norms for coding agents, including conditional classification of application interface surfaces and execution environments.
