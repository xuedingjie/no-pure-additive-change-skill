# Repository Agent Rules

## No Pure Additive Change

When a requirement adds or changes a database field, table, index, status, enum, type, channel, source, callback, push, sync job, batch job, statistic, provider payload, persistent business state, or business branch, the agent must not default to pure additive implementation.

Before implementation, use `skills/no-pure-additive-change/SKILL.md`.

The agent must produce a design judgment before editing code. The judgment must compare at least:

1. Minimal additive implementation
2. Local model correction or evolutionary refactor

Pure additive implementation is a fallback, not the default. It is allowed only when the agent proves it does not duplicate state, split source of truth, create another future field/table/branch, leak provider-specific data into a core model, worsen a high-risk service, or hide migration/query/index risk.

Database persistence has priority over code assumptions, but persisted data is usable as evidence only when it is proven valid for the business question. The agent must verify the authoritative table/field, write path, read path, effective-record filters, field semantics, status semantics when relevant, verification path when relevant, indexes, transaction boundary, and idempotency or locking strategy. Do not treat table existence, field existence, or row existence as sufficient evidence.

If the affected fact involves persistent business state, external events, provider callback, or state change, the agent must explicitly exclude records that should not participate in the current business judgment, such as deleted, failed, test, historical, temporary, or migrated records. If these filters or verification paths are unclear, stop before implementation and mark the gap as an assumption requiring confirmation.

If the touched model already contains additive debt of the same kind, the implementation must include a local correction option. Leaving the debt untouched requires explicit justification.

Any refactor, migration plan, business-impacting change, or proposal that changes a model boundary must include evidence lines. If data is migrated or reinterpreted, include migration tests. If behavior is refactored, include business regression tests and business invariants. Unsupported claims must be marked as assumptions.

Project-specific review evidence for this rule should be maintained as Markdown under `skills/no-pure-additive-change/references/`. If table volume, risk level, core table lists, source-of-truth notes, high-risk service lists, field/status semantics, or effective-record filters change, update the Markdown evidence and this rule/skill together.
