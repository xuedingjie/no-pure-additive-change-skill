# No Pure Additive Change Skill

**Language**: [中文](README.md) | English

This repository contains a generic rule + skill package for AI coding agents.

It is intended for backend systems where new feature requests often become pure additive changes: adding one more field, table, enum, status, branch, callback, job, statistic, or provider-specific payload without checking whether the model should be corrected.

This repository is not tied to any business domain. It does not assume any industry-specific model. It only provides a generic workflow for identifying and constraining pure additive implementation of new requirements.

The goal is not broad refactoring. The goal is to force the agent to make an evidence-backed design judgment before implementation:

- If minimal additive implementation is acceptable, prove it will not create new debt.
- If the touched model is already weak, consider local model correction instead of another field/table/branch.
- If data migrates, provide a migration plan and migration tests.
- If behavior is refactored, provide business regression tests and business invariants.
- If races are possible, design for multiple instances or pods.

## What It Enforces

- Pure additive implementation is a fallback, not the default.
- The agent must compare minimal additive implementation with local model correction or evolutionary refactor.
- Claims must include evidence lines; unsupported claims must be marked as assumptions.
- Persisted database facts have priority over code assumptions.
- Persisted data is valid evidence only after field semantics, effective-record filters, status semantics, write/read paths, indexes, verification paths, and concurrency behavior are checked.
- Data migration requires a migration test plan.
- Behavioral refactor requires business regression tests and business invariants.
- Race-sensitive logic must be designed for multi-instance or multi-pod deployment.

<details>
<summary><strong>Click to expand: Valid database evidence checklist</strong></summary>

The agent must explain:

- authoritative table and field
- business meaning of the field
- unit, precision, value range, or enum meaning when relevant
- status semantics when relevant
- valid record filters
- deleted, failed, test, historical, temporary, and migrated record exclusions when relevant
- write path, read path, and derivation path
- unique key, idempotency key, or natural key
- indexes used by reads, writes, migration, and verification
- transaction boundary
- correctness under multiple instances or pods
- verification path against authoritative, derived, or provider records

If these cannot be answered, the design is not ready for implementation.

</details>

## Files

| File | Purpose |
|---|---|
| `AGENTS.md` | Repository-level rule that can be copied into a project |
| `skills/no-pure-additive-change/SKILL.md` | Reusable agent skill |
| `skills/no-pure-additive-change/references/context-map-template.md` | Template for project-specific risk maps and source-of-truth notes |
| `skills/no-pure-additive-change/references/pressure-scenarios.md` | Generic failure scenarios the skill is designed to block |
| `README.md` | Chinese README |

## How To Use

1. Copy `AGENTS.md` into the target repository or merge its rule into existing agent instructions.
2. Copy `skills/no-pure-additive-change/` into the target repository or agent skill directory.
3. Fill `references/context-map-template.md` with project-specific core tables, high-risk services, source-of-truth rules, valid record filters, and business test focus.
4. Require agents to read `skills/no-pure-additive-change/SKILL.md` before implementing changes that affect persistent state or model boundaries.

## Project Calibration Required

The skill is generic, but risk judgment must be calibrated per project.

Before relying on it in a real codebase, maintainers should fill in:

- P0/P1 tables
- high-risk services or modules
- source-of-truth map
- field and status semantics
- effective-record filters
- verification paths
- migration and rollback conventions
- deployment model and concurrency assumptions

Do not treat this package as a substitute for business evidence. It forces the agent to find evidence, compare options, expose assumptions, and stop when evidence is insufficient.

## License

This project is licensed under the [MIT License](LICENSE).
