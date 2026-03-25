# Skill: Test-Driven Development

## When this skill activates
Activates automatically when implementing any new function, class, API endpoint,
or feature. Do not wait to be asked — this is the default workflow.

## Workflow

**Step 1 — Write the failing test first.**
Before writing a single line of implementation code:
- Write a test that describes the expected behavior
- Run the test and confirm it FAILS with a meaningful error (not a syntax error)
- If the test passes before implementation exists, the test is wrong — rewrite it

**Step 2 — Write the minimum implementation to make it pass.**
- Write only what's needed to make the specific test pass
- Do not add functionality not covered by the test
- Run the test and confirm it PASSES

**Step 3 — Refactor only after green.**
- Clean up the implementation without changing behavior
- Run the test again after refactoring — confirm still green
- Then add the next test

## Rules

- Never write implementation before a failing test exists. No exceptions.
- If the codebase already has a test file for this module, add to it. Don't create
  a separate test file.
- Use the test framework already in the project (read `package.json`, `pyproject.toml`,
  or `.claude/project-config.json` to identify it). Don't introduce a new framework.
- Test names should describe behavior, not implementation:
  - Good: `should return 401 when token is expired`
  - Bad: `test_jwt_validator_line_47`

## Reference
Read `.claude/memory/conventions.md` for any project-specific testing patterns
that have been learned in previous sessions.
