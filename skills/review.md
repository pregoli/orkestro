# Review Skill

Guidelines for reviewing code changes.

## Review Process

1. **Read the full diff** before commenting on anything
2. **Understand intent** — what was the change trying to accomplish?
3. **Check each file** against the checklist below
4. **Classify every finding** — blocker, warning, or nit
5. **Verify test coverage** — missing tests is always a blocker

## Review Checklist

### Correctness
- Does the code do what it's supposed to?
- Are edge cases handled (nulls, empty collections, boundary values)?
- Are error paths handled (exceptions, timeouts, invalid input)?
- Is the logic correct for concurrent/async scenarios?

### Conventions
- Does it follow the existing patterns in the codebase?
- Are naming conventions consistent (casing, prefixes, suffixes)?
- Is the file in the right directory?
- Does it follow the project's architectural layering?

### Test Coverage
- Is every new code path covered by a test?
- Are both happy path and error paths tested?
- Are edge cases tested?
- Do tests assert the right things (not just "no exception thrown")?

### Safety
- No hardcoded secrets, API keys, or passwords
- No SQL injection, XSS, or other injection vulnerabilities
- No unsafe deserialization or file path manipulation
- Sensitive data is not logged or exposed in error messages

### Maintainability
- Is the code readable without extensive comments?
- Are functions focused (single responsibility)?
- Are there any obvious performance issues?
- Could this change break other parts of the system?

## Severity Levels

| Level | Meaning | Blocks merge? |
|---|---|---|
| **Blocker** | Bug, security issue, missing tests, breaks production | Yes |
| **Warning** | Convention violation, potential issue, should improve | No (but should fix) |
| **Nit** | Style, readability, minor suggestion | No |

## Scope Rule

Review ONLY the changed lines. Pre-existing issues in unchanged code:
- Note as "Pre-existing: description"
- Never count as blockers
- Never trigger REQUEST_CHANGES on their own

## Verdict Rules

- `APPROVE` — no blockers found
- `REQUEST_CHANGES` — one or more blockers found
- Warnings alone do NOT justify REQUEST_CHANGES
- Missing test coverage is always a blocker
