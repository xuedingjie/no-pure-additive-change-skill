# No Pure Additive Change Skill

This repository contains a generic rule + skill package for AI coding agents.

It is intended for backend systems where feature requests often become pure additive changes: adding one more field, table, enum, status, branch, callback, job, or provider-specific payload without checking whether the model should be corrected.

## What It Enforces

- Pure additive implementation is a fallback, not the default.
- The agent must compare minimal additive implementation with local model correction or evolutionary refactor.
- Persisted database facts have priority over code assumptions.
- Persisted data is valid evidence only after effective-record filters, amount semantics, status semantics, write/read paths, indexes, reconciliation, and concurrency behavior are checked.
- Data migration requires a migration test plan.
- Behavioral refactor requires business regression tests and business invariants.
- Race-sensitive logic must be designed for multi-instance or multi-pod deployment.
- Claims must include evidence lines; unsupported claims must be marked as assumptions.

## Files

- `AGENTS.md`: repository-level rule that can be copied into a project.
- `skills/no-pure-additive-change/SKILL.md`: reusable agent skill.
- `skills/no-pure-additive-change/references/context-map-template.md`: template for project-specific risk maps and source-of-truth notes.
- `skills/no-pure-additive-change/references/pressure-scenarios.md`: generic failure scenarios the skill is designed to block.

## How To Use

1. Copy `AGENTS.md` into the target repository or merge its rule into the existing agent instructions.
2. Copy `skills/no-pure-additive-change/` into the target repository or agent skill directory.
3. Fill `references/context-map-template.md` with project-specific core tables, high-risk services, source-of-truth rules, and business test focus.
4. Require agents to read `skills/no-pure-additive-change/SKILL.md` before implementing changes that affect persistent state or model boundaries.

## Project Calibration Required

The skill is generic, but the risk judgment must be calibrated per project.

Before relying on it in a real codebase, maintainers should fill in:

- P0/P1 tables
- high-risk services or modules
- source-of-truth map
- amount/status semantics
- effective-record filters
- reconciliation paths
- migration and rollback conventions
- deployment model and concurrency assumptions

Do not treat this package as a substitute for business evidence.
