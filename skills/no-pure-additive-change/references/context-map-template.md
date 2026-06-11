# Context Map Template

Use this file to calibrate the generic skill to a specific project.

Do not leave placeholders for high-risk areas. If the project cannot identify source of truth, valid data filters, field/status semantics, or high-risk tables/services, the agent must treat related changes as not ready for implementation.

## P0 Tables

P0 tables reject pure additive implementation by default.

| table | type | approximate rows | size | why high risk |
|---|---|---:|---:|---|
| example_core_record | core aggregate |  |  | current business state |
| example_external_event | integration event |  |  | external event side effects |
| example_audit_history | audit/history |  |  | historical traceability |

## P1 Tables

P1 tables require explicit comparison between minimal additive implementation and local model correction.

| table | type | why important |
|---|---|---|
| example_provider_callback | provider event | raw provider payload and retry handling |

## High-Risk Services

| service/module/class | risk level | why high risk |
|---|---|---|
| ExampleCoreService | P0 | owns current state and many workflow branches |
| ExampleEventService | P0 | writes integration or event side effects |

## Source Of Truth Map

### Current State

- Authoritative table/field:
- Derived/read model:
- Historical/event table:
- Valid record filters:
- Change rule:

### Derived Values And Business Attributes

- Authoritative table/field:
- Event/log table:
- Field meaning:
- Unit/precision/value range, if relevant:
- Valid record filters:
- Verification path:
- Change rule:

### State Flow And Operation History

- Current-state table:
- Event/history table:
- Provider/external-state table:
- Valid record filters:
- Change rule:

### Provider Callback, Push, Sync

- Raw provider payload table:
- Normalized business conclusion:
- Idempotency key:
- Retry strategy:
- Valid record filters:
- Change rule:

## Business Test Focus

All business domains must first confirm valid persisted database facts:

- authoritative table/field
- write path
- read path
- unique/idempotency key
- indexes
- transaction boundary
- effective-record filters
- field semantics and value range when relevant
- status semantics when relevant
- verification path when relevant

If callbacks, retries, scheduled jobs, duplicate requests, or concurrent state changes are involved, assume multiple service instances or pods and cover idempotency and race conditions.

### Example Domain: Core Workflow

- create/cancel/update/detail/list/export
- state and sub-state transitions
- operation logs and state-flow records
- forbidden operations under abnormal statuses

### Example Domain: Derived Or Stored Attributes

- field creation/update/backfill
- compatible reads during migration
- derived value consistency
- duplicate event handling and verification

### Example Domain: Provider Integration

- successful callback, failed callback, duplicate callback, out-of-order callback, unknown status
- signature/decryption failure, retry, raw record preservation, normalized business state

## Update Rules

Update this context map when:

- P0/P1 table list changes
- high-risk service list changes
- source-of-truth ownership changes
- valid record filters change
- field semantics, value range, or verification rules change
- status semantics or current/history/provider-state ownership changes
- new core tables, extension tables, state-flow tables, or provider record tables are introduced
- production table volume crosses an order of magnitude
