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
   with [TASK-CODE] — <task description>. TDD cycle: RED."

Task format in SPEC.md (tasks alternate TEST-IMPL within each phase):
- `- [ ] [TEST-XX-01] Write tests for X in tests/file.test.ts`
- `- [ ] [IMPL-XX-02] Implement X in src/file.ts -> satisfies [TEST-XX-01]`
- `- [ ] [TEST-XX-03] Write tests for Y in tests/file.test.ts`
- `- [ ] [IMPL-XX-04] Implement Y in src/file.ts -> satisfies [TEST-XX-03]`

For each TEST-IMPL pair in scope, in order:

### RED — TEST Task

1. Mark the task with `<- current` in the SPEC.md
2. Read the task specification: file path, test descriptions, isolation strategy
3. Write the test file at the specified path
4. Write test code with the assertions described in the spec:
   - Set up test fixtures, mocks, or containers as specified
   - Write each test case described in the task
   - Import the production module/function (which does not exist yet or is empty)
   - Assert expected behavior
5. **GATE: RUN the tests** via Bash (e.g., `npx vitest run <file>`,
   `pytest <file>`, `go test ./...`). Do not proceed without running them.
6. **GATE: Confirm tests FAIL.** If tests fail — good, this is RED.
7. If tests PASS unexpectedly: **STOP.** Do not proceed. Report the anomaly
   to the user. Possible causes:
   - The feature already exists (check the codebase)
   - The test doesn't actually test what it claims (assertions are wrong)
   - Imports resolve to existing code that satisfies the test
   - The test file has a syntax/import error that makes the runner skip it
   Wait for the user to decide how to proceed.
8. Log the failure output in the TDD Log:
   `| [TEST-XX-NN] | <command>: N tests, N failed — <key message> | — | — |`
9. Check off the task: `- [ ]` -> `- [x]`, remove `<- current`

### GREEN — IMPL Task

1. Mark the task with `<- current` in the SPEC.md
2. Read which TEST task this satisfies (from `-> satisfies [TEST-XX-NN]`)
3. **Read the test file.** Understand exactly what the tests expect — return
   values, status codes, error messages, data shapes. The tests are the spec.
4. Write the **MINIMUM** production code to make the referenced tests pass:
   - Do not add features beyond what the tests require
   - Do not add error handling the tests don't check for
   - Do not optimize prematurely
   - Write the simplest thing that could possibly work
5. **GATE: RUN the tests** via Bash. Do not proceed without running them.
6. **GATE: Confirm tests PASS.**
7. If tests still fail:
   - Read the failure output carefully
   - **Fix the production code, NOT the tests.** Tests define expected
     behavior. If a test expects X and your code does Y, your code is wrong.
   - Run again. Repeat until all referenced tests are green.
   - The only reason to touch a test is if it has an actual bug (wrong
     import, syntax error, broken setup). If you believe the test expectation
     itself is wrong, STOP and ask the user before changing it.
8. Log the pass output in the TDD Log:
   `| [IMPL-XX-NN] | — | <command>: N passed, 0 failed | — |`

### REFACTOR — Still on IMPL Task

1. Review the implementation for code quality:
   - Remove duplication
   - Improve naming
   - Extract helpers if warranted
   - Simplify complex logic
   - Ensure code follows existing codebase patterns
2. **GATE: RUN tests again** after refactoring. Do not proceed without running.
3. **GATE: Confirm tests STILL PASS.**
4. If refactoring broke tests: undo the refactoring change, try a different
   approach, run tests again. Do not modify tests to accommodate refactoring.
5. Log refactoring in the TDD Log Refactor column (or "none")
6. Check off the task: `- [ ]` -> `- [x]`, remove `<- current`
7. Update spec progress (checkboxes, phase status, registry, Resume Context)

**Then move to the next TEST-IMPL pair and repeat the cycle.**

### Self-Check (run this before every task)

Before starting any task, verify:
- Am I about to write production code? → Is there a failing test for it?
- Am I about to skip running tests? → Run them. Always. Via Bash.
- Am I about to modify a test to make it pass? → Stop. Fix the code instead.
- Did I log the last task in the TDD Log? → Do it now.
- Did I update the checkbox and `<- current` marker? → Do it now.

If any check fails, correct it before proceeding.

### Tests Are Sacred

Tests define expected behavior. They are the specification in code form.

- **Never modify a test assertion to match what your code returns.** Make
  your code return what the test expects.
- If a test expects status 422 and your code returns 400, change your code
  to return 422. Do not change the test to expect 400.
- The only time you touch a test is for actual bugs: wrong import, syntax
  error, broken fixture setup — not because the assertion "doesn't match."
- If you genuinely believe a test expectation is wrong (contradicts
  requirements, impossible to implement), **STOP and ask the user.** Do not
  silently change test expectations.

### Blocking Rule

Each IMPL task cannot start until its TEST task is completed and tests are
confirmed failing. This is per-task, not per-phase. If you find yourself
about to write a function body before its test exists, STOP and write the
test first.

If the user asks to skip ahead to an IMPL task whose TEST task is not done,
explain the constraint and offer to write the test first.

### Test Execution Rule

Run the actual test command via Bash at every RED, GREEN, and REFACTOR
transition. That is **3 runs minimum per TEST-IMPL cycle.** Never say "tests
would pass" or "this should work." Execute the tests and report the actual
output. If the test runner is not available, report the blocker immediately.

Run the specific test file(s) for the current task, not the entire suite
(unless the user requests it or the task requires it).

### Phase Transition

Phases group by feature, not by test vs implementation. When all tasks
(both TEST and IMPL) in a phase are done, the phase is `[completed]` and
the next phase becomes `[in-progress]`.

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
