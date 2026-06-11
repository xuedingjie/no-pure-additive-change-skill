---
name: no-pure-additive-change
description: Use when a new backend requirement adds or changes fields, tables, statuses, enums, channels, provider data, callbacks, pushes, jobs, statistics, indexes, business branches, persistent state, or model boundaries in a codebase with additive-change risk
---

# No Pure Additive Change

## Core Principle

Pure additive implementation is technical debt unless proven harmless.

When a requirement touches an already weak model, duplicated state, unclear source of truth, provider-specific leakage, or an overloaded service, use the requirement as a narrow opportunity to correct the touched model.

For low-risk areas, local model correction is preferred over minimal additive implementation when it removes duplication, clarifies source of truth, separates provider-specific data, or prevents the next similar field/table/branch.

The goal is not broad refactoring. The goal is bounded, evidence-backed model correction with migration, rollback, migration tests, and business tests.

Design must prioritize valid persisted database facts over transient application assumptions. A database fact is not valid merely because a table, field, or row exists. Whether the recommendation is minimal additive or refactor, treat database rows, constraints, indexes, write paths, verification queries, and data validity conditions as the primary evidence of business truth. Then evaluate execution efficiency.

When the change can involve races, retries, callbacks, scheduled jobs, duplicate requests, or concurrent state changes, assume the service may be deployed as multiple instances or pods. Do not rely on in-memory locks, local caches, static variables, or single-process ordering for correctness.

## Evidence To Read First

Read project-specific Markdown references before making a design judgment.

Recommended reference files:

- `skills/no-pure-additive-change/references/context-map-template.md`
- `skills/no-pure-additive-change/references/pressure-scenarios.md`

The canonical review context should be Markdown so it can be reviewed, versioned, and updated with the rule and skill.

## Trigger

Use this skill when the request mentions or implies:

- add field, add table, add status, add enum, add type, add source, add channel
- add callback, push, sync, batch job, scheduled job, statistic, cache, report
- add provider-specific request/response/result fields
- change persistent business state
- change lifecycle, external integration, audit/history, callback, or state-flow logic
- add another branch to an existing service
- add another table/field because the current model does not fit
- split, merge, rename, migrate, reinterpret, or backfill data

Do not use this skill for isolated copy changes, formatting-only edits, local test fixture changes, or bug fixes that do not affect persistent state, business process, model boundaries, or service responsibilities.

## Evidence Lines Required

Any design judgment, refactor proposal, migration plan, migration test plan, or business test plan must include evidence lines.

An evidence line must include:

- source type: code, metadata, test, log, documentation, or assumption
- file or reference document
- line number when available
- observed fact
- why it matters

Unsupported claims must be marked as assumptions.

If an assumption affects source of truth, persistent state, status, migration, rollback, or business correctness, stop and ask for confirmation.

Evidence must prioritize database persistence facts:

- table and field where the business fact is stored
- business meaning of the field, including field semantics, status semantics when relevant, value range when relevant, and valid record conditions
- unique key, idempotency key, or natural key
- indexes used by reads, writes, consistency checks, or migration
- insert/update/delete paths
- queries that derive current state or historical facts
- filters that distinguish valid, deleted, failed, test, historical, temporary, or migrated records
- consistency or verification evidence for authoritative, derived, and provider data
- transaction boundary and lock/compare-and-set behavior when relevant

Code evidence is still required, but code behavior must be checked against persisted database facts.

For persisted fields, statuses, and external events, evidence must also answer:

- Is the field meaning clear enough for the current requirement?
- Is the field value range, unit, precision, or enum mapping clear when relevant?
- Is the status meaning and ownership clear when relevant?
- Are invalid, deleted, failed, test, historical, temporary, or migrated records excluded correctly when relevant?
- Can the result be verified against authoritative, derived, or provider records?

## Evidence Levels

| Level | Meaning |
|---|---|
| E1 | direct code evidence |
| E2 | metadata or context-map evidence |
| E3 | existing test/log evidence |
| E4 | naming/comment inference |
| A | assumption requiring confirmation |

Example:

```markdown
**Evidence Lines**
- E1 `module/path/LifecycleService.java:123`: this method reads persisted status and decides the next workflow step, so the change affects state judgment.
- E2 `references/context-map.md:P0 Tables`: `core_record` is marked as a P0 source-of-truth table, so field changes require migration and business regression tests.
- E4 `column_name=provider_result`: the field name indicates provider-specific semantics, but no business normalization logic was found.
- A Assumption: `status=3` means closed; business confirmation is required.
```

## Required Context Pass

1. Search the codebase for the business concept, table names, fields, model classes, mapper/DAO usage, service methods, jobs, callbacks, and provider boundaries.
2. Identify touched tables and classify their risk from the project context map.
3. Identify touched services and classify their risk from the project context map.
4. Check the source-of-truth section in the project context map for the relevant business concept.
5. Inspect code around entity/model, mapper/DAO, SQL, repository/accessor, service, controller, job, facade/client, and provider integration boundaries.
6. Identify the database write path, read path, unique/idempotency key, index usage, and transaction boundary for the touched business fact.
7. Identify whether the database evidence is valid for the business question: field semantics, status semantics, effective record filters, deleted/failed/test/historical/temporary/migrated data, and verification path.
8. If concurrency is possible, identify whether correctness holds across multiple instances or pods.

Use semantic search only when it is available. Otherwise, use exact search.

Fallback exact-search workflow:

```bash
# table / field / model
rg "table_name|field_name|EntityName|ModelName" <repo-root>

# SQL and persistence usage
rg "table_name|field_name|MapperName|DaoName|RepositoryName|select|update|insert" <repo-root>

# business branch and status usage
rg "Status|status|subStatus|type|channel|source|provider" <repo-root>

# service entry points and callers
rg "ServiceName|methodName|ControllerName|JobName|FacadeName|ClientName" <repo-root>
```

## Decision Rule

1. If the request touches a P0 table or P0 service:
   - pure additive is rejected by default
   - recommend controlled evolutionary refactor or staged redesign
   - allow additive only as a compatibility layer with explicit evidence

2. If the request touches P1:
   - compare minimal additive, local model correction, and controlled refactor
   - prefer model correction when it removes duplicated state or branching

3. If the request touches P2/P3:
   - prefer low-risk local correction when model smell exists
   - minimal additive is acceptable only when no model smell exists

4. If the same kind of field, branch, status, provider field, or table already exists:
   - do not add the N+1 instance by default
   - model the pattern instead

5. If migration, business tests, or evidence lines cannot be described:
   - the design is not ready

## Option Set

### Option A: Minimal Additive

A fallback, not the default.

Allowed only when:

- no duplicated concept exists
- no source of truth is split
- persisted database fact, validity conditions, and write path are clear
- no model smell exists
- no high-risk table or service is worsened
- query and write efficiency are acceptable
- migration/backfill/rollback are trivial
- future similar requirements are unlikely to create another field/table/branch

### Option B: Local Model Correction

Preferred for low-risk touched models when pure additive would worsen debt.

May include:

- rename or clarify model fields with compatibility
- split extension/detail/relation tables
- extract status/event records
- move provider-specific data out of core model
- extract enum, helper, handler, pusher, strategy, policy, or processor
- separate current state from historical event

Must include:

- evidence lines
- database fact and efficiency analysis
- migration plan if data changes
- migration tests if data migrates
- business tests for affected behavior

### Option C: Controlled Evolutionary Refactor

Use for P0/P1 areas when additive change would worsen source-of-truth, state, provider-boundary, or service responsibility problems.

Must be bounded and compatible.

### Option D: Staged Domain Redesign

Use only when current model is actively blocking correctness, scalability, or maintainability.

Requires:

- staged rollout
- dual-write or read compatibility if needed
- backfill
- consistency checks
- rollback
- multi-instance race and idempotency design when concurrent writes/events are possible
- business regression test matrix

## Model Smells That Force Refactor Consideration

If any smell appears in the touched area, Minimal Additive cannot be recommended without stronger proof:

- another similar field already exists
- status/type/source/channel appears in multiple places without clear ownership
- provider/channel/product-specific field is being added to a core table
- nullable fields encode workflow stages
- JSON/string fields hide structured business state
- service branches by provider/channel/type/status
- field meaning depends on comments or caller convention
- current state and historical events are mixed
- a new requirement would likely add the same kind of field again later

## Migration Requires Tests

Any data migration, backfill, dual-write, read compatibility, field split, table split, status mapping, derived value recalculation, provider data relocation, or source-of-truth change must include a migration test plan.

A migration plan without tests is incomplete.

Migration and backfill design must also state how it avoids long locks, full-table scans in online paths, write amplification, and unsafe retries on multi-instance deployments.

Migration plan template:

```markdown
**Migration Plan**
- Current data shape:
- Target data shape:
- Migration approach:
- Dual-write needed:
- Read compatibility needed:
- Backfill SQL / job:
- Idempotency key:
- Batch strategy:
- Verification SQL:
- Rollback:
- Old field/table cleanup timing:
```

Migration test plan template:

```markdown
**Migration Test Plan**
- Mapping tests:
- null/default/unknown tests:
- Consistency verification tests:
- Idempotency tests:
- Compatible read tests:
- Rollback tests:
- Dirty data tests:
- Verification SQL:
- Automated test location:
- Manual acceptance steps:
```

## Refactor Requires Business Tests

Any non-pure-additive implementation must include a business test plan.

If the change refactors model, table, state, persisted field, callback, job, service, handler, pusher, strategy, policy, processor, or source-of-truth logic, the design is incomplete until it specifies:

1. migration tests, if data moves or is reinterpreted
2. business regression tests for affected user/system workflows
3. business invariant tests for rules that must never break

Behavior-preserving refactors should start with characterization tests where practical.

Business tests must include concurrency/idempotency cases when the touched workflow can be executed by multiple instances, repeated callbacks, retries, scheduled jobs, or concurrent user actions.

Business test plan template:

```markdown
**Business Test Plan**
- Affected business workflows:
- Behaviors that must not change:
- Business invariants:
- Positive cases:
- Negative/forbidden cases:
- Boundary status cases:
- Idempotency/repeated request cases:
- Provider callback cases:
- Concurrency/retry cases:
- Query/export/report cases:
- Automated test location:
- Manual acceptance path:
```

Business test matrix:

```markdown
| Test Type | Business Scenario | Evidence Line | Initial Data | Operation | Expected Result | Verification Point | Automated Location |
|---|---|---|---|---|---|---|---|
| Positive flow | ... | E1/E2 | ... | ... | ... | ... | ... |
| Forbidden flow | ... | E1/E2 | ... | ... | ... | ... | ... |
| Boundary status | ... | E1/E2 | ... | ... | ... | ... | ... |
| Idempotency | ... | E1/E2 | ... | ... | ... | ... | ... |
| Compatibility | ... | E1/E2 | ... | ... | ... | ... | ... |
```

## Multi-Instance Concurrency Rules

Assume multiple service instances or pods unless deployment evidence proves otherwise.

For race-sensitive changes, the design must explain:

- idempotency key or unique constraint
- transaction boundary
- optimistic lock, compare-and-set update, or database lock strategy
- duplicate callback/request handling
- retry behavior
- out-of-order event handling
- scheduled job overlap protection
- why the chosen index supports the concurrency path efficiently

Do not rely on:

- in-memory locks
- static maps or local caches as correctness source
- process-local ordering
- single-instance assumptions
- non-atomic read-then-write sequences without a database guard

## Business Invariants

For every refactor, list the business invariants that must remain true.

Examples:

- the same external event must not create duplicate business effects
- current state and latest state-change event cannot conflict without documented reason
- invalid or deleted records must not be treated as active business facts
- provider raw callback record must be preserved when the project requires auditing
- rollback must not lose original persisted data needed for recovery

## Service Rules

If the change touches a high-risk service, do not add new branches by default.

Use these extraction rules:

| Symptom | Preferred shape |
|---|---|
| callback updates multiple aggregates | `CallbackHandler` |
| external push with retry/result recording | `Pusher` |
| reusable preparation/filter/mapping | `Helper` |
| behavior varies by provider/channel/product/status | `Strategy` or `Policy` |
| batch job mixes query, mutation, external call | `Processor` plus small helpers |

## Design Judgment Template

Before editing code, output this block:

```markdown
### No Pure Additive Design Judgment

**One-line Judgment**
- ...

**Triggers Hit**
- Tables:
- Code:
- Business concept:

**Evidence Lines**
- Code evidence:
  - E1 ...
- Data/metadata evidence:
  - E2 ...
- Test/log evidence:
  - E3 ...
- Naming/comment inference:
  - E4 ...
- Assumptions:
  - A ...

**Source Of Truth**
- Current authority:
- Persisted database fact:
- Data validity conditions:
- Status semantics, if relevant:
- Field semantics/value range, if relevant:
- Verification path:
- Write path:
- Read/derivation path:
- Whether this creates a second truth source:
- Unknowns:

**Why Pure Additive Is Not Default**
- ...

**Model Smells**
- ...

**Option Comparison**
1. Minimal Additive:
   - Approach:
   - Benefit:
   - Risk:
   - Why acceptable or unacceptable:
2. Local Model Correction:
   - Approach:
   - Benefit:
   - Risk:
   - Migration/compatibility:
3. Controlled Evolutionary Refactor:
   - Approach:
   - Benefit:
   - Risk:
4. Staged Redesign, if relevant:
   - Approach:
   - Benefit:
   - Risk:

**Recommended Plan**
- ...

**Data Impact**
- Fields:
- Tables:
- Indexes:
- Unique/idempotency keys:
- Transaction boundary:
- Query/write efficiency:
- Effective-record filters:
- Field semantics/value range:
- Business consistency verification:
- null/default:
- backfill:
- rollback:

**Concurrency And Multi-Instance**
- Race exists:
- Idempotency strategy:
- Lock/optimistic lock/conditional update:
- Retry and out-of-order handling:
- Scheduled job overlap:
- Correctness evidence under multiple instances:

**Migration Plan**
- Not involved / involved:
- ...

**Migration Test Plan**
- Not involved / involved:
- ...

**Business Test Plan**
- Affected workflows:
- Business invariants:
- Test matrix:
- Automated tests:
- Manual acceptance:

**Code Impact**
- Service:
- Helper/Pusher/Handler/Strategy/Policy/Processor:
- Integration/Facade/Client:
- Controller/Job:
- Tests:

**First Verification Point**
- ...

**Falsifier**
- If ... is found, this plan is invalid and must change to ...
```

For trivial changes, keep the block short but still state why simple additive implementation is acceptable.

## Hard Stops

Stop before implementation if:

- source of truth is unclear
- persisted database fact is unclear
- persisted database fact is not proven valid for the business question
- effective-record filters are unclear for deleted, failed, test, historical, temporary, or migrated data
- field semantics, value range, unit, precision, or enum mapping is unclear when relevant
- status semantics or ownership is unclear when relevant
- verification path is unclear for authoritative, derived, or provider facts
- target table risk is unknown
- target service risk is unknown
- claim has no evidence line
- assumption affects persistent state, status, migration, rollback, or source of truth
- query/write efficiency is not evaluated for affected database paths
- race-sensitive change has no multi-instance idempotency or locking design
- migration exists but migration test plan is missing
- refactor exists but business test plan is missing
- business invariants are not listed
- behavior-preserving refactor has no baseline or characterization test plan
- rollback cannot be described

## Completion Check

Before claiming ready to implement:

- design judgment block is present
- evidence lines are present and unsupported claims are assumptions
- touched tables and services have risk levels
- source of truth is identified
- persisted database fact is proven valid for the business question
- write path, read path, effective-record filters, and efficiency are addressed
- field semantics, value range, status semantics, and verification path are addressed when relevant
- multi-instance race handling is addressed when concurrency is possible
- minimal additive and local correction/evolutionary refactor are compared
- migration plan exists when data moves or is reinterpreted
- migration tests exist when migration exists
- business tests exist when behavior is refactored
- business invariants are listed
- rollback and verification commands are named
