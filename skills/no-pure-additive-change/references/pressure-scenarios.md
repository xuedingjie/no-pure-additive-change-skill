# Pressure Scenarios

This file records generic failure scenarios the skill must block. It is not a business case library.

## Baseline Failure Patterns

| Scenario | Common output without the skill | Must block |
|---|---|---|
| Add a field to a core table | Directly change entity, mapper, API response | No source-of-truth check, no history/default/null strategy |
| Add a money amount field | Add field and display logic only | No unit, precision, sign, idempotency, rollback, reconciliation |
| Add a state/status | Add enum and if-else branch | No distinction between current state, historical event, failed attempt, provider state |
| Add a provider callback field | Put provider-specific field into core table | Provider leakage into core model |
| Modify a large service | Add one more branch | High-risk service continues growing and side effects may be missed |
| Add list filter/report | Add unindexed query or low-selectivity index | No large-table efficiency analysis or pagination strategy |
| Propose migration | Describe backfill only | No mapping, reconciliation, idempotency, compatibility, rollback tests |
| Propose refactor | Describe code structure only | No business regression tests or business invariants |
| Give refactor conclusion | No evidence lines | Cannot review fact, inference, and assumption separately |
| No semantic search tool exists | Skip context investigation | Must use exact search fallback for fields, tables, persistence, services, jobs, callbacks, provider boundaries |
| Only code is inspected | Infer business truth from service branches or DTO fields | Must confirm database facts, write/read paths, keys, indexes, transaction boundary |
| Only table/field existence is inspected | Treat all persisted rows as business truth | Must confirm effective-record filters, amount unit/precision/sign, status semantics, deleted/failed/reversed/test/migrated exclusion, reconciliation path |
| Race-sensitive logic is designed as single-process | Use in-memory lock, static map, local cache, process-local ordering | Must design idempotency, unique constraints, locks, conditional updates, retries, out-of-order handling for multiple instances |

## Skill Must Force

- Read Markdown context evidence before making a judgment.
- Compare minimal additive implementation with local model correction or evolutionary refactor.
- Reject pure additive by default for P0 tables and high-risk services.
- Identify source of truth.
- Prioritize valid persisted database facts and evaluate query/write efficiency.
- Do not treat table existence, field existence, or row existence as sufficient business evidence.
- State effective-record filters and exclude deleted, failed, reversed, voided, test, historical, temporary, and migrated data when they should not participate.
- State amount unit, precision, sign direction, status semantics, and reconciliation path when relevant.
- Assume multiple instances or pods for races, callbacks, retries, scheduled jobs, and duplicate requests.
- Explain 10x data impact, migration, rollback, and index impact.
- Provide evidence lines; unsupported claims must be assumptions.
- Provide migration tests when data migrates.
- Provide business regression tests and business invariants when behavior is refactored.
- If choosing simple additive implementation, prove it will not create a second truth source or future repeated field/table/branch.
