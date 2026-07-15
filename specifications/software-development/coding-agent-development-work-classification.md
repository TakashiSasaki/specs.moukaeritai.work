# Development Work Classification and Change Governance for Coding Agents

- **Status:** Draft
- **Version:** 0.1.0
- **Date:** 2026-07-15
- **Intended audience:** Coding agents, maintainers, reviewers, and repository owners

## 1. Purpose

This specification defines how coding agents MUST classify development work, select the correct source of truth, place artifacts, preserve compatibility, and choose validation appropriate to the change.

Its central purpose is to prevent several distinct architectural dimensions from being collapsed into a single directory hierarchy or naming convention.

A coding agent MUST distinguish at least the following questions:

- What subject or domain is being changed?
- What kind of artifact is being changed?
- What implementation responsibility does the artifact perform?
- What role does the artifact play in defining or demonstrating correctness?
- What is the intended change, and what must remain unchanged?
- Does the change affect an externally visible interface?
- If so, who is the external audience and by what technical modality do they interact?

The goal is not classification for its own sake. The goal is to make the following explicit:

> What is changing, what must be preserved, which canonical source governs the change, who is affected, which boundary must enforce correctness, and what evidence is required.

## 2. Normative language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as normative requirement levels.

- **MUST / REQUIRED** indicates an unconditional requirement.
- **MUST NOT** indicates an unconditional prohibition.
- **SHOULD** indicates a default requirement from which deviation requires a concrete reason.
- **SHOULD NOT** indicates a discouraged practice that requires justification.
- **MAY** indicates an optional practice.

## 3. Core principle: independent classification axes

Development work MUST NOT be described using only one architectural axis.

The following statements are foundational:

```text
interface surface != repository structure
interface modality != interface surface
artifact type != implementation responsibility
normative specification != implementation
validation logic != evidence of validation
path name != security boundary
```

A file, module, endpoint, schema, or test MAY participate in several dimensions simultaneously. A coding agent MUST NOT force these dimensions into a false one-to-one correspondence.

## 4. Required classification axes

Before changing a repository, a coding agent MUST determine the following five axes.

| Axis | Governing question | Typical values |
|---|---|---|
| Subject or domain | What is the change about? | object, marker, identity, routing, authentication, deployment, repository governance |
| Artifact type | What kind of artifact is being changed? | specification, documentation, source code, test, fixture, configuration, script, generated artifact |
| Implementation responsibility | What does it do internally? | presentation, boundary, application, domain, validation, infrastructure, persistence, coordination |
| Normative role | What position does it occupy relative to correctness? | normative, implementation, validation, evidence, explanation, coordination |
| Change intent | What is intended to change, and what must be preserved? | feature, fix, refactor, compatibility change, migration, deprecation, removal, documentation, generated alignment |

The classification MAY remain implicit for a small and obvious change, but it MUST be reflected in implementation choices, validation, and the pull request or work report.

## 5. Conditional classification axes

When relevant, a coding agent MUST also determine the following axes.

| Axis | Governing question | When required |
|---|---|---|
| Interface surface | Who interacts with the system, and for what purpose? | When externally visible behavior is affected |
| Interface modality | Through what technical form does interaction occur? | For user, system, or process boundaries |
| Execution environment and operational phase | Where and when is the artifact used? | When behavior differs by environment or lifecycle phase |
| Provenance and canonicality | Is this canonical, derived, generated, vendored, cached, or historical? | When information is duplicated or generated |
| Compatibility and stability | Who depends on the current behavior or representation? | For public interfaces, stored data, automation, or versioned artifacts |
| Trust and security boundary | What is trusted, and where are validation and authorization enforced? | For external input, privileges, secrets, or production-sensitive operations |
| Lifecycle state | Is the item proposed, experimental, active, deprecated, retired, or archived? | For long-lived specifications and features |
| Ownership and governance | Who may approve or maintain it? | When multiple owners or teams participate |

## 6. Interface surfaces are audience-oriented

### 6.1 Definition

An interface surface is an externally observable or operable part of a system, classified primarily by the external audience and that audience's purpose.

The standard audience-oriented surfaces are:

| Surface | Primary audience | Primary purpose |
|---|---|---|
| `public` | Anonymous visitors and the general public | Introduction, discovery, onboarding, authentication entry, and public information |
| `user` | Normal authenticated users | The system's primary user work |
| `operator` | Administrators and operational staff | Administration, auditing, controlled maintenance, repair, and operations |
| `maintainer` | Developers and maintainers of the system itself | Diagnostics, configuration inspection, implementation explanation, and maintenance support |
| `integrator` | Developers or systems integrating with the system | System-to-system integration |
| `verifier` | Testers and verification actors | Controlled execution, observation, and verification |

Existing systems MAY continue to use established aliases when renaming would create unnecessary disruption:

```text
app   ~= user
admin ~= operator
dev   ~= maintainer
test  ~= verifier
```

A broad rename MUST NOT be performed solely for cosmetic conformity.

### 6.2 Applicability

A surface classification applies when an external interface is affected.

A shared domain model, reusable validator, persistence adapter, build script, or CI workflow MUST NOT be assigned to exactly one surface merely because one surface currently uses it.

One internal module MAY support several surfaces.

### 6.3 Surface is not access policy

Audience classification and access control are separate concerns.

For example, an `integrator` interface may be public, authenticated by API credentials, or restricted to privileged clients. A `maintainer` interface may be read-only or may perform production-sensitive operations.

A surface name MUST NOT be treated as proof of authorization.

## 7. Interface modalities are separate from surfaces

Web pages, APIs, command-line interfaces, files, libraries, events, and webhooks describe how interaction occurs. They do not by themselves identify the intended audience.

Standard modality values include:

```text
web
api
cli
file
event
webhook
library
```

Examples:

```yaml
surface: integrator
modality: api
```

```yaml
surface: maintainer
modality: cli
```

```yaml
surface: verifier
modality: web
```

An API MUST NOT automatically be classified as an integrator surface. APIs may serve normal users, operators, maintainers, integrators, or verification tools.

## 8. Artifact types and repository placement

Repository structure SHOULD primarily organize artifacts by artifact type, build unit, implementation responsibility, or domain cohesion. It SHOULD NOT mechanically mirror externally visible URL or command structures.

Typical artifact categories include:

```text
specifications/  normative requirements and machine-readable definitions
docs/            explanatory material, rationale, guides, and procedures
src/             application implementation
functions/       server-side or deployed function implementation
packages/        reusable models, validators, serializers, and libraries
tests/           executable verification
fixtures/        controlled inputs and expected outputs
scripts/         generation, validation, maintenance, and coordination tools
generated/       outputs derived from canonical sources
```

A project MAY use different names, but the responsibility of each location MUST be clear.

The following are different artifacts even when they describe the same capability:

```text
/api/v1/...                         executable API endpoint
specifications/api/v1/...          normative API specification
docs/api/...                        explanatory API documentation
functions/src/boundaries/api/...    API boundary implementation
tests/api/...                       API verification
```

## 9. Specifications, documentation, and contracts

### 9.1 Specifications

A specification defines requirements that an implementation or externally visible interface is expected to satisfy.

Specifications MAY include:

- domain and data models;
- JSON Schema, XML Schema, OpenAPI, or equivalent formal definitions;
- data exchange protocols;
- API requirements;
- identifier and naming rules;
- routing conventions;
- security requirements;
- coding conventions;
- repository structure rules;
- compatibility, deprecation, and migration rules.

### 9.2 Documentation

Documentation primarily explains rationale, background, use, operation, or examples.

The default distinction is:

```text
specifications define what is required
documentation explains why, how, and when
```

A human-readable Markdown file MAY be normative, but its normative status MUST be explicit.

### 9.3 Contracts

A contract is a specification that defines a commitment across a boundary between independently changing actors, components, services, or versions.

Appropriate contract examples include:

- public request and response formats;
- event or webhook formats;
- externally visible identifier formats;
- component-to-component messages;
- compatibility guarantees for versioned exchange formats.

General coding conventions, architectural guidance, and explanatory documents SHOULD NOT all be described collectively as contracts.

The conceptual relationship is:

```text
specifications
└── contracts
```

A migration from an existing `contracts/` hierarchy to a broader `specifications/` hierarchy MUST be treated as a separate compatibility-aware change. Canonical references, generators, registries, historical versions, and validation scripts MUST be reviewed before moving files.

## 10. Implementation responsibilities

Implementation code SHOULD be classified by responsibility when doing so improves dependency control and reviewability.

Standard responsibility categories include:

| Responsibility | Purpose |
|---|---|
| Presentation | Render and collect user-facing information |
| Boundary | Parse and adapt external input and output |
| Application | Coordinate use cases and transactions |
| Domain | Express domain concepts, invariants, and state transitions |
| Validation | Determine conformance to a representation or rule set |
| Infrastructure | Integrate runtime services and technical facilities |
| Persistence | Store and retrieve state |
| Coordination | Build, release, CI, migration, and repository automation |

External entry points SHOULD remain thin and delegate reusable behavior to application and domain layers.

```text
web / api / cli / file / test boundary
                    |
                    v
            application service
                    |
                    v
                 domain
                    |
                    v
       infrastructure / persistence
```

Business logic MUST NOT be duplicated independently for each surface or modality when the same use case is being performed.

## 11. Normative roles

Each artifact SHOULD have a clear primary role relative to correctness.

| Role | Meaning |
|---|---|
| `normative` | Defines what is correct or required |
| `implementation` | Realizes required behavior |
| `validation` | Determines whether input or output conforms |
| `evidence` | Demonstrates that conformance or behavior was observed |
| `explanation` | Explains rationale, context, or usage |
| `coordination` | Controls development, release, migration, or operational procedure |

Typical classifications include:

```text
JSON Schema       normative
validator code    validation and implementation
application code  implementation
unit test         evidence
API guide         explanation
GitHub workflow   coordination
```

A test MUST NOT silently become the only specification when a normative rule is required. Tests are normally evidence of conformance to a specification or explicitly documented behavior.

## 12. Canonicality and provenance

When the same information exists in more than one representation, the coding agent MUST identify the canonical source before editing.

Standard provenance values include:

```text
canonical
derived
generated
vendored
cached
snapshot
legacy
```

Generated artifacts MUST NOT be edited as though they were canonical. The canonical source MUST be changed and the generation process rerun.

When possible, types, validators, reference documentation, and compatibility tests SHOULD be generated from or checked against one canonical schema or model.

Historical and legacy artifacts MUST NOT be modified merely to make them resemble current active artifacts unless the repository explicitly defines them as mutable.

## 13. Data exchange models, schemas, and validators

### 13.1 Normative models and schemas

Normative data exchange models, schemas, protocols, and compatibility requirements SHOULD be placed under a specification hierarchy, for example:

```text
specifications/
├── domain/
├── data-exchange/
│   ├── models/
│   ├── schemas/
│   ├── protocols/
│   ├── compatibility/
│   └── examples/
└── api/
    └── v1/
```

A format used across several modalities or surfaces SHOULD NOT be placed under an API-only hierarchy.

### 13.2 Executable validators

Executable validators belong with reusable implementation code, for example:

```text
packages/data-exchange/
├── types/
├── validators/
├── serializers/
└── generated/
```

When practical, structural validators SHOULD be generated from canonical schemas.

### 13.3 Validation layers

A coding agent MUST distinguish the following layers:

1. **Syntax validation** — whether the representation can be parsed and uses the expected encoding or content type.
2. **Structural validation** — required fields, types, enumerations, additional properties, and schema conformance.
3. **Semantic validation** — domain relationships, state transitions, identity consistency, and business invariants.
4. **Authentication and authorization** — who the actor is and whether the actor may perform the operation.

These layers MAY be invoked in one request pipeline, but they MUST NOT be conceptually collapsed into one undifferentiated validator.

Database lookup, domain state validation, and authorization MUST NOT be hidden inside a schema validator without an explicit architectural reason.

## 14. Change intent and compatibility

A coding agent MUST state or infer what behavior is intended to change and what behavior is intended to remain stable.

Standard change intents include:

```text
feature
fix
refactor
compatibility
migration
deprecation
removal
documentation
generated-alignment
```

A refactor SHOULD preserve externally observable behavior, persistent data representations, and published contracts unless the change explicitly states otherwise.

Before modifying a URL, command, API, exchange format, stored representation, or published identifier, the coding agent MUST consider:

- existing users and clients;
- external references and bookmarks;
- automation and scripts;
- stored data;
- versioning policy;
- compatibility aliases;
- redirects;
- deprecation periods;
- migration procedures.

Broad renaming or relocation MUST NOT be performed solely to make names cosmetically conform to a new convention.

## 15. Execution environment and operational phase

A coding agent MUST distinguish where and when an artifact is used when that distinction affects safety or behavior.

Typical values include:

```text
local
build-time
CI
test
staging
production
runtime
deployment-time
migration-time
```

Development and verification capabilities such as identity impersonation, arbitrary fixture loading, fault injection, unrestricted internal-state inspection, or production data mutation MUST NOT be made unconditionally available in production.

When feasible, dangerous maintainer and verifier interfaces SHOULD be excluded from production route or command registration rather than merely hidden from navigation.

## 16. Trust and security boundaries

Names and paths communicate purpose but do not enforce security.

At an external boundary, the implementation MUST independently consider:

```text
parsing
structural validation
authentication
authorization
semantic validation
rate limiting
auditing
persistence
```

External input MUST be treated as untrusted regardless of surface or modality.

A route under `/admin`, `/dev`, or `/test` MUST NOT be considered protected merely because of its path.

## 17. Relationship between surfaces and repository structure

The repository structure MUST NOT be forced to match the surface structure.

The desired relationship is traceability, not identity:

```text
surface structure != repository structure
surface structure <-> repository structure
```

When useful, a project SHOULD maintain a mapping among runtime entry points, normative specifications, implementation modules, documentation, and verification assets.

| Surface | Runtime entry point | Normative specification | Implementation | Verification |
|---|---|---|---|---|
| public | public routes or commands | public behavior and routing requirements | public entry adapters | public-access and routing tests |
| user | normal application workflows | domain and user workflow requirements | features and use cases | user-flow tests |
| operator | administrative interfaces | administration, audit, and safety requirements | operator boundaries | authorization and audit tests |
| maintainer | diagnostics and maintenance interfaces | development and maintenance requirements | maintenance tools | environment-restriction tests |
| integrator | API, file, event, or library interfaces | integration and exchange specifications | integration boundaries | compatibility tests |
| verifier | test harnesses and scenario controls | verification requirements | harnesses and scenario runners | tests of the harness itself |

## 18. Web path structure is a derived specification

For a web application, an audience-oriented surface model can be mapped to a coherent URL path convention. That mapping is useful, but it is not the definition of the surfaces themselves.

A web path guideline SHOULD therefore be maintained as a separate, derived specification.

A suitable companion location is:

```text
specifications/interfaces/web/web-application-surface-paths.md
```

That companion specification may define preferred path roots, exceptions, redirects, versioned machine endpoints, and production restrictions.

The companion specification MUST preserve the following distinctions:

- A URL prefix is a web-specific projection of a surface, not the surface itself.
- Repository directories do not need to match URL prefixes.
- Domain-oriented routes may belong semantically to a surface without being nested under its preferred prefix.
- Paths do not provide authorization.
- Existing stable URLs must be evaluated for compatibility before renaming.
- Machine-facing API paths must be distinguished from human-facing documentation and specification publication paths.

This document intentionally does not define the final URL vocabulary. It defines the meta-level rules under which that vocabulary is to be designed.

## 19. Coding-agent workflow

Before implementing a change, a coding agent MUST perform the following reasoning sequence:

1. Identify the subject or domain.
2. Identify the artifact type.
3. Identify the implementation responsibility.
4. Identify the normative role.
5. State what is intended to change and what must remain stable.
6. Determine whether an external interface is affected.
7. If an external interface is affected, identify the audience surface and technical modality.
8. Identify the canonical source and any generated or derived artifacts.
9. Evaluate compatibility, lifecycle, environment, and security implications.
10. Select validation that provides evidence appropriate to the affected dimensions.

The coding agent MUST read repository-specific instructions and canonical-source declarations before modifying files.

The coding agent MUST prefer the smallest coherent change that satisfies the intended behavior. It MUST NOT perform unrelated restructuring merely because a broader taxonomy exists.

## 20. Validation selection

Validation MUST be selected according to the affected classification axes and risk, not merely according to the number of changed files.

Examples:

- A normative schema change requires schema validation, compatibility checks, and generated-artifact alignment.
- An implementation change requires relevant unit or integration tests, type checking, and build verification.
- An external surface change requires routing, authorization, compatibility, and user- or client-flow checks.
- A generated artifact change requires proof that it matches its canonical source.
- A security-boundary change requires negative tests for unauthenticated, unauthorized, spoofed, and cross-owner access.
- An environment-sensitive change requires evidence that production restrictions remain effective.

A small textual change MAY require broad validation if it changes a canonical specification or coordination rule.

## 21. Pull request and work-report requirements

A pull request or work report SHOULD state:

1. What changed.
2. What intentionally did not change.
3. The affected subject or domain.
4. The artifact types changed.
5. The change intent.
6. The canonical source affected.
7. Any effect on external surfaces or interface modalities.
8. Compatibility, security, or environment implications.
9. Validation performed.
10. Migration, deprecation, or follow-up work.

When there is no external-interface effect, the report SHOULD say so explicitly.

Example:

```text
This change modifies internal validation implementation only.
It does not change the integrator surface, exchange schema,
persisted representation, or externally observable compatibility.
```

## 22. Prohibited practices

A coding agent MUST NOT:

- treat API as an audience surface without identifying the actual audience;
- treat specifications as equivalent to the maintainer surface;
- use `contracts` as an unrestricted synonym for all specifications and documentation;
- mechanically mirror external URLs in repository directories;
- directly edit generated artifacts as canonical sources;
- use tests as the sole specification when a normative rule is required;
- mix schema validation, domain rules, database state, and authorization without explicit boundaries;
- treat a route or directory name as an authorization mechanism;
- perform broad moves or renames solely for cosmetic conformity;
- update canonical, generated, derived, and historical artifacts without distinguishing their roles;
- duplicate the same business rule independently across surfaces or modalities;
- introduce a new abstraction or hierarchy without a concrete current requirement.

## 23. Decision procedure

When uncertain, a coding agent SHOULD answer these questions in order:

1. Is this externally visible?
2. Who uses it, and for what purpose?
3. Through what modality do they interact?
4. What is the canonical source?
5. Is the artifact normative, implementation, validation, evidence, explanation, or coordination?
6. What internal responsibility is changing?
7. What must remain compatible?
8. Which environment and trust boundaries are involved?
9. What evidence demonstrates correctness?

## 24. Classification examples

### 24.1 External integration request schema

```yaml
domain: association
surface: integrator
modality: api
artifact: specification
responsibility: boundary
normative_role: normative
intent: additive-feature
environment: runtime
provenance: canonical
compatibility: backward-compatible
security_boundary: authenticated-external-client
lifecycle: active
```

### 24.2 Validator generated from the schema

```yaml
domain: association
surface: integrator
modality: api
artifact: generated-code
responsibility: validation
normative_role: validation
intent: generated-alignment
environment: runtime
provenance: generated
compatibility: inherited-from-schema
```

### 24.3 Compatibility test

```yaml
domain: association
surface: integrator
modality: api
artifact: test
responsibility: validation
normative_role: evidence
intent: preserve-compatibility
environment: CI
provenance: derived
```

### 24.4 Internal routing diagnostics page

```yaml
domain: routing
surface: maintainer
modality: web
artifact: source-code
responsibility: presentation
normative_role: implementation
intent: diagnostic-feature
environment: non-production-or-restricted-production
security_boundary: privileged-maintainer
```

## 25. Conformance

A change conforms to this specification when:

- the relevant classification axes have been considered;
- canonical and generated artifacts have been distinguished;
- external audience and modality have not been conflated;
- the implementation respects responsibility and trust boundaries;
- compatibility and change intent are explicit where relevant;
- validation provides appropriate evidence;
- unrelated restructuring has been avoided.

Conformance does not require every repository to use identical directory names or to record every classification axis as metadata.

## 26. Change history

### 0.1.0 — 2026-07-15

- Initial draft.
- Defines independent classification axes for development work.
- Establishes audience-oriented interface surfaces and separate interface modalities.
- Distinguishes specifications, contracts, documentation, implementation, validation, and evidence.
- Defines placement principles for data exchange models, schemas, and validators.
- Establishes web path structure as a separate derived specification.
