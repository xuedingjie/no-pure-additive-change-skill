# Context Map Template

Use this file to calibrate the generic skill to a specific project.

Do not leave placeholders for high-risk areas. If the project cannot identify source of truth, valid data filters, amount/status semantics, or high-risk tables/services, the agent must treat related changes as not ready for implementation.

## P0 Tables

P0 tables reject pure additive implementation by default.

| table | type | approximate rows | size | why high risk |
|---|---|---:|---:|---|
| example_order | core aggregate |  |  | current business state |
| example_bill | money/accounting |  |  | bill authority |
| example_payment_log | money/event log |  |  | payment side effects |

## P1 Tables

P1 tables require explicit comparison between minimal additive implementation and local model correction.

| table | type | why important |
|---|---|---|
| example_provider_callback | provider event | raw provider payload and retry handling |

## High-Risk Services

| service/module/class | risk level | why high risk |
|---|---|---|
| ExampleOrderService | P0 | owns current state and many workflow branches |
| ExamplePaymentService | P0 | writes money-related side effects |

## Source Of Truth Map

### Current State

- Authoritative table/field:
- Derived/read model:
- Historical/event table:
- Valid record filters:
- Change rule:

### Money, Balance, Billing, Payment, Refund, Settlement

- Authoritative table/field:
- Event/log table:
- Amount unit:
- Precision/rounding:
- Sign direction:
- Valid record filters:
- Reconciliation path:
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
- amount unit/precision/sign when relevant
- status semantics when relevant
- reconciliation path when relevant

If callbacks, retries, scheduled jobs, duplicate requests, or concurrent state changes are involved, assume multiple service instances or pods and cover idempotency and race conditions.

### Example Domain: Order

- create/cancel/update/detail/list/export
- current state and sub-state transitions
- operation logs and state-flow records
- forbidden operations under abnormal statuses

### Example Domain: Billing And Payment

- bill generation/close/overdue/prepayment
- payment callback/refund/retry
- account flow and payment log side effects
- duplicate payment, duplicate callback, amount unit, precision, sign, reconciliation

### Example Domain: Provider Integration

- successful callback, failed callback, duplicate callback, out-of-order callback, unknown status
- signature/decryption failure, retry, raw record preservation, normalized business state

## Update Rules

Update this context map when:

- P0/P1 table list changes
- high-risk service list changes
- source-of-truth ownership changes
- valid record filters change
- amount unit, precision, sign direction, or reconciliation rules change
- status semantics or current/history/provider-state ownership changes
- new core tables, extension tables, state-flow tables, or provider record tables are introduced
- production table volume crosses an order of magnitude
