---
description: Implement the active spec using strict TDD — red-green-refactor for every task
disable-model-invocation: true
---

# Implement Spec (TDD)

Implement tasks from the active spec using strict test-driven development.
See also the specsmith-tdd skill (SKILL.md) for global invariants and
`references/spec-format.md` for the SPEC.md template.

User's request: $ARGUMENTS

## Scope Detection

Parse the user's request to determine what to implement:

- **No argument / "the spec" / "implement"** -> Start from the `<- current`
  task and work forward through all remaining tasks
- **"phase N" / "phase <name>"** -> Implement all unchecked tasks in that
  specific phase only
- **"all phases" / "everything"** -> Implement all remaining unchecked tasks
  across all phases, in order
- **"task CODE-NN"** -> Implement just that specific task

## Implementation Workflow

1. Read `.specs/registry.md` to find the spec with `active` status
2. If none is active, show the user their specs so they can choose one
3. Load `.specs/<id>/SPEC.md`
4. Parse all phases and tasks — identify which tasks are in scope
5. Determine task type by task code prefix: `TEST-` for test tasks,
   `IMPL-` for implementation tasks
6. Present a brief plan: "I'll implement N tasks across M phases. Starting
   with [TASK-CODE] — <task description>. TDD phase: RED/GREEN."

Task format in SPEC.md:
- `- [ ] [TEST-XX-01] Write tests for X in tests/file.test.ts — assertions: Y, Z (mock: HTTP client, real: database)`
- `- [ ] [IMPL-XX-02] Implement X in src/file.ts -> satisfies [TEST-XX-01]`

For each task in scope, in order:

### For TEST Tasks (RED Phase)

1. Mark the task with `<- current` in the SPEC.md
2. Read the task specification: file path, test descriptions, isolation strategy
3. Write the test file at the specified path
4. Write test code with the assertions described in the spec:
   - Set up test fixtures, mocks, or containers as specified
   - Write each test case described in the task
   - Import the production module/function (which does not exist yet or is empty)
   - Assert expected behavior
5. **RUN the tests** via Bash (e.g., `npx vitest run <file>`, `pytest <file>`,
   `go test ./...`, `mvn test -pl <module>`)
6. **Confirm tests FAIL** — this is the RED phase. The tests must fail because
   the production code does not exist yet.
7. If tests PASS unexpectedly: **STOP**. Report the anomaly to the user.
   Possible causes:
   - The feature already exists (check the codebase)
   - The test doesn't actually test what it claims (assertions are wrong)
   - Imports resolve to existing code that satisfies the test
   - The test file has a syntax/import error that makes the runner skip it
   Do not proceed until the user clarifies.
8. Log the failure output in the TDD Log:
   - Task column: task code
   - Red column: test command, exit code, number of failures, key failure message
9. Check off the task: `- [ ]` -> `- [x]`
10. Remove `<- current` from the completed task
11. Update spec progress (checkboxes, phase status, registry, Resume Context)

### For IMPL Tasks (GREEN Phase)

1. Mark the task with `<- current` in the SPEC.md
2. Read which TEST tasks this satisfies (from `-> satisfies [TEST-XX-NN]`)
3. Locate the test files for those TEST tasks — read them to understand
   exactly what the tests expect
4. Write the **MINIMUM** production code to make the referenced tests pass:
   - Do not add features beyond what the tests require
   - Do not add error handling the tests don't check for
   - Do not optimize prematurely
   - Write the simplest thing that could possibly work
5. **RUN the tests** via Bash — run the specific test files referenced by
   the satisfies notation
6. **Confirm tests PASS** — this is the GREEN phase
7. If tests still fail:
   - Read the failure output carefully
   - Fix the production code (not the tests, unless the test has a bug)
   - Run again
   - Repeat until all referenced tests pass
   - Do not check off the task until all referenced tests are green
8. Log the pass output in the TDD Log:
   - Task column: task code
   - Green column: test command, exit code, number passing, time elapsed
9. **REFACTOR phase**: Review the implementation for code quality:
   - Remove duplication
   - Improve naming
   - Extract helpers if warranted
   - Simplify complex logic
   - Ensure code follows existing codebase patterns
10. **RUN tests again** after refactoring — they must still pass
11. If refactoring broke tests: undo the refactoring change, try a different
    approach, run tests again
12. Log refactoring in the TDD Log:
    - Refactor column: what changed (or "none" if no refactoring needed)
13. Check off the task: `- [ ]` -> `- [x]`
14. Remove `<- current` from the completed task
15. Update spec progress (checkboxes, phase status, registry, Resume Context)

### Blocking Rule

**You MUST NOT write production code for any implementation task until the
corresponding test task(s) are completed and their tests are confirmed
failing.** If you find yourself about to write a function body before its
test exists, STOP and write the test first.

If the user asks to skip ahead to an IMPL task whose TEST tasks are not yet
done, explain the TDD constraint and offer to write the tests first.

### Test Execution Rule

**You MUST run the actual test command** at every red-green-refactor
transition. Do not say "tests would pass" or "this should work." Execute the
tests and read the output. If the test runner is not configured or not
available, report the blocker and ask the user how to run tests.

Always run the specific test file(s) relevant to the current task, not the
entire test suite (unless the user requests it or the task requires it).

### Phase Transition

- When all TEST tasks in a test phase are done: phase -> `[completed]`,
  advance the next IMPL phase to `[in-progress]`
- When all IMPL tasks in an impl phase are done: phase -> `[completed]`,
  advance the next TEST phase to `[in-progress]`
- Test phases and impl phases are paired — completing a TEST phase unlocks
  its IMPL phase, and completing an IMPL phase unlocks the next TEST phase

### Update Progress (after every task)

1. Check off the task: `- [ ]` -> `- [x]`
2. Remove `<- current` from the completed task
3. Move `<- current` to the next unchecked task (or the next in-scope task)
4. If all tasks in the current phase are done:
   - Phase status: `[in-progress]` -> `[completed]`
   - Next phase (if any): `[pending]` -> `[in-progress]`
5. Update the `updated` date in YAML frontmatter
6. Recompute progress count (X/Y) from checkboxes
7. Update progress and `updated` date in `.specs/registry.md`
8. Update Resume Context with current state, including TDD phase

## Handle Issues

- If a task is more complex than expected, split it into subtasks and update
  the SPEC.md before continuing
- If implementation diverges from the spec (better approach found, errors in
  spec, etc.), log it in the **Deviations** section
- Log any new technical decisions in the **Decision Log**
- If blocked on a task:
  - Keep the task unchecked and add a blocker note in Resume Context
  - Set the phase marker to `[blocked]` only when the whole phase is blocked
  - Move to the next unblocked task if possible
- If a test is flaky (passes sometimes, fails sometimes): flag it immediately,
  investigate the cause, fix the non-determinism before proceeding

## Completion

When all in-scope tasks are done:

- If all tasks in the spec are complete:
  - Set all phases to `[completed]`
  - Set spec status to `completed` in frontmatter
  - Update `.specs/registry.md` with `completed` status
  - Run the full test suite one final time to confirm everything passes
  - Present a summary: tasks completed, files created/modified, tests
    passing, coverage if available
  - Suggest next spec to activate if any are paused
- If only a phase was completed:
  - Report phase completion and remaining work
  - Set the next phase to `[in-progress]` if applicable
  - State whether the next phase is TEST or IMPL

## Quality Standards

During implementation:
- Write clean, simple, maintainable code — no over-engineering
- Follow existing codebase patterns and conventions
- Use the libraries specified in the spec's Library Choices section
- Handle edge cases identified in the spec
- Validate at system boundaries, trust internal code

TDD-specific quality:
- Every test must be isolated — no shared mutable state between tests
- No external network calls in unit tests — mock at boundaries
- Use testcontainers for real databases/services when specified (not
  in-memory substitutes unless the spec explicitly calls for them)
- Mock only at boundaries (external APIs, third-party services) — do not
  mock internal modules or functions
- Tests must be deterministic — no time-dependent, order-dependent, or
  race-condition-prone patterns
- Test names must describe the behavior, not the implementation:
  "returns 401 when token is expired" not "test verifyToken method"
- Each test should have a single reason to fail
