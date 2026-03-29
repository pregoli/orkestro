# Planning Skill

Guidelines for producing safe, sequenced implementation plans.

## Task Sequencing

Always sequence work in this order to avoid inconsistent states:

1. **Core logic first** — domain models, business rules, data structures
2. **Infrastructure second** — database schemas, repositories, external integrations
3. **Interface third** — API endpoints, UI components, CLI commands
4. **Tests throughout** — every code change must have a corresponding test

This order ensures:
- Core logic exists before anything references it
- Infrastructure supports the data before interfaces expose it
- Interfaces only expose functionality that is fully wired

## Task Atomicity

One task = one file. Never bundle multiple files into a single task.

If a change requires touching 5 files, that is 5 tasks. This enables:
- Parallel execution of independent tasks
- Clear progress tracking
- Easier rollback if a single task fails

## Dependency Tracking

Every task has a `Depends On` field listing which tasks must complete first:
- Tasks with `Depends On = none` can run immediately
- Tasks with the same dependencies can run in parallel
- This enables wave-based execution: Wave 1 (no deps) → Wave 2 (depends on Wave 1) → etc.

## Risk Assessment

For each change, assess:

| Change Type | Risk Level | Mitigation |
|---|---|---|
| New file | Safe — fully additive | None needed |
| New property/field (optional) | Safe if nullable or has default | Add default value |
| Modify existing interface | Caution — callers may break | Check all callers first |
| Remove field/property | Breaking — dependents will fail | Two-phase: deprecate then remove |
| Change data schema | Breaking — existing data may be invalid | Migration script required |

## Anti-Patterns

Avoid these common planning mistakes:
- Planning interface changes before core logic exists
- Forgetting to plan test files
- Bundling multiple files into one task
- Leaving design decisions unresolved ("clarify with team later")
- Planning a migration without checking existing data compatibility
