# Development Skill

Guidelines for implementing code changes safely and correctly.

## Read Before Write

Before modifying any file, read it in full. Understand:
- The existing structure and patterns
- How the code you're changing is used by other parts of the system
- What conventions the file follows (naming, formatting, error handling)

Never rely on summaries or assumptions about file contents.

## Implementation Checklist

For every change:

- [ ] Read the file you're modifying
- [ ] Follow existing patterns in the codebase (don't introduce new conventions)
- [ ] Handle error cases (null checks, validation, exception handling)
- [ ] Use the project's existing utilities and helpers — don't reinvent
- [ ] Keep changes minimal — implement what the plan asks, nothing more

## Build Verification

After implementing changes:
1. Run the project's build command
2. If it fails, read the errors carefully and fix them
3. Do not move on until the build passes

## Test Verification

After the build passes:
1. Run the project's test suite
2. If tests fail, determine whether the failure is in your code or the test
3. Fix the issue and re-run
4. Do not move on until all tests pass

## Code Quality

- **No dead code** — don't leave commented-out code or unused imports
- **No hardcoded values** — use constants or configuration
- **No secrets** — never commit API keys, passwords, or tokens
- **Consistent style** — match the existing codebase, not your personal preference
- **Meaningful names** — variables, functions, and files should describe what they do

## What NOT to Do

- Don't refactor code you weren't asked to change
- Don't add features beyond what the plan specifies
- Don't change formatting or style in files you're not modifying
- Don't add comments explaining obvious code
- Don't introduce new dependencies without explicit need
