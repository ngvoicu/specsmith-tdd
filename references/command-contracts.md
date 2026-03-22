# Command Contracts

This file defines functional contracts for `commands/*.md` and the universal
`SKILL.md` behavior. Use it as a review checklist before releases.

## Global Contracts

1. `SPEC.md` frontmatter is authoritative; registry is denormalized.
2. Exactly one active spec should exist after any write operation.
3. All write workflows update `SPEC.md` first, then recompute/update registry.
4. Forge workflow never writes application code.
5. Phase markers use `[pending]`, `[in-progress]`, `[completed]`, `[blocked]`.
   Task lines use checkboxes and `<- current`; tasks are not tagged `[blocked]`.

## Command Contracts

### `/specsmith-tdd:forge`

1. Resolve `<spec-id>` before research output paths are referenced.
2. Collision-check existing spec IDs before creating new files.
3. Forge must not run in plan mode; if plan mode is active, require exit before
   continuing (Claude Code only — other tools proceed normally).
4. Create `.specs/<spec-id>/` directory before spawning the researcher or
   writing any research output.
5. Output scope is `.specs/` artifacts only (`research-*.md`, `interview-*.md`,
   `SPEC.md`, `registry.md` updates).
6. After approval, handoff to `/specsmith-tdd:implement` instead of implementing
   inside forge.

### `/specsmith-tdd:implement`

1. Supports scope parsing: current flow, phase-specific, all phases, task code.
2. For each completed task: checkbox + current marker + phase markers +
   frontmatter `updated` + registry progress/date.
3. Blocked handling:
   - Keep blocked tasks unchecked.
   - Mark phase `[blocked]` only when the phase is blocked.
   - Record blocker context in Resume Context/Decision Log/Deviations as needed.

### `/specsmith-tdd:resume`

1. If no active spec exists, list specs and request target.
2. Include progress, current phase/task, and Resume Context in output.

### `/specsmith-tdd:pause`

1. If no active spec exists, report no-op and stop.
2. Persist detailed Resume Context with concrete file/function references.
3. Set status `paused` and sync registry.

### `/specsmith-tdd:switch`

1. Validate target ID and target `SPEC.md` existence before pausing current spec.
2. If target already active, report and stop.
3. Pause current (if any), activate target, resume target, sync registry.

### `/specsmith-tdd:list`

1. Handle missing registry gracefully.
2. Group by status in order: active, paused, completed, archived.
3. If `SPEC.md` missing for a row, keep row visible and flag it.

### `/specsmith-tdd:status`

1. Show detailed phase/task breakdown for active spec.
2. If no active spec, guide to activate one.

### `/specsmith-tdd:openapi`

1. Generate/update `.openapi/openapi.yaml` and `.openapi/endpoints/*.md`.
2. Preserve manual additions when updating existing files.
3. Report endpoint/schema/security counts and manual-review candidates.

## Universal Skill Contract

1. `SKILL.md` must include cross-tool behavior for all declared triggers.
2. If `generate openapi` is listed as a trigger, OpenAPI workflow behavior must
   be defined in `SKILL.md` (not only plugin command files).
3. Command-specific docs can specialize behavior but cannot violate critical
   invariants from `SKILL.md`.
4. `SKILL.md` must be self-contained for standalone users (`npx skills add`).
   Any content essential for spec creation must be inlined (e.g., the SPEC.md
   template skeleton). References to `references/*.md` and `commands/*.md`
   should be conditional ("Plugin users: see...").
5. Agent spawning (researcher) must have a graceful fallback for tools that
   don't support agents.

## TDD Contracts

These contracts enforce test-driven development discipline across all commands
and the universal skill. They are **non-negotiable** — violations break the
TDD guarantee.

### Global TDD Contracts

1. **No production code without a failing test.** Every IMPL task must have a
   corresponding TEST task that was written and executed first. If no failing
   test exists for the code being written, stop and write the test.

2. **Tests MUST be executed.** Writing a test file is not enough. The test
   runner must be invoked and the output (pass/fail counts, failure messages)
   must be recorded in the TDD Log.

3. **TDD Log updated after every task.** After completing any TEST or IMPL
   task, add a row to the TDD Log with the red/green/refactor outputs.

4. **TEST phases precede IMPL phases.** In the spec phase structure, every
   feature's TEST phase comes before its IMPL phase. Phase ordering is:
   `Tests for X`, then `Implement X`, then `Tests for Y`, then `Implement Y`.

5. **Continuous task numbering with TEST-/IMPL- prefixes.** Task codes
   increment across all phases: `[TEST-XX-01]`, `[TEST-XX-02]`,
   `[TEST-XX-03]`, `[IMPL-XX-04]`, `[IMPL-XX-05]`, etc. No resets per phase.

6. **IMPL tasks reference their TEST tasks.** Every IMPL task line includes
   `-> satisfies [TEST-XX-NN]` linking to the test(s) it makes pass.

7. **Refactor only when green.** Refactoring (renaming, extracting, restructuring)
   happens only after all tests pass. Refactoring must not change test outcomes.

8. **Test isolation.** Each test is independent. No test relies on another test's
   side effects. Tests can run in any order and produce the same results.

### Forge TDD Contracts

9. **Testing Architecture section required.** Every spec produced by forge MUST
   include the Testing Architecture section (framework, tools, isolation
   strategy, coverage targets, test commands, anti-patterns). Specs without
   this section are incomplete.

10. **Testing interview questions.** During the interview phase, forge MUST ask
    at least 2 questions about testing preferences:
    - Preferred test framework and runner (or detect from project)
    - Integration test strategy (Testcontainers, test DB, mocks)
    - Coverage expectations
    - Any existing testing patterns to follow

11. **Test-interleaved phase structure.** Forge MUST produce phases in
    TEST/IMPL pairs. A phase list like `Phase 1: Build X`, `Phase 2: Build Y`
    is invalid. The correct structure is `Phase 1: Tests for X (TEST)`,
    `Phase 2: Implement X (IMPL)`, `Phase 3: Tests for Y (TEST)`,
    `Phase 4: Implement Y (IMPL)`.

12. **Research includes test infrastructure.** The researcher agent MUST
    analyze the project's existing test infrastructure (framework, patterns,
    coverage tools, anti-patterns) as part of the research phase.

### Implement TDD Contracts

13. **Red-green-refactor enforcement.** During implementation, the workflow for
    each feature is:
    - Execute TEST phase tasks: write tests, run them, confirm they fail (RED)
    - Execute IMPL phase tasks: write code, run tests, confirm they pass (GREEN)
    - Refactor: improve code while keeping tests green (REFACTOR)

14. **Blocking rule: TEST phase must complete before IMPL phase starts.** The
    implement command MUST NOT begin an IMPL phase until its corresponding TEST
    phase is `[completed]`. If a user requests "implement phase 4" and phase 3
    (its TEST phase) is not complete, refuse and explain.

15. **Test execution mandate.** The implement command MUST run the test suite
    after every task completion. The exact command from the spec's Testing
    Architecture section is used. Test output is captured and recorded.

16. **Failed tests block progress.** If tests fail after an IMPL task (and the
    failures are not expected RED-phase failures), implementation stops. The
    failure must be resolved before moving to the next task.

17. **TDD Log is mandatory.** The implement command MUST update the TDD Log
    table after each task. Missing TDD Log entries indicate the TDD process
    was not followed.

### Resume TDD Contracts

18. **TDD phase indicator.** Resume output MUST include the current TDD phase
    (RED, GREEN, or REFACTOR) and list any currently failing tests. This
    information comes from the Resume Context section of the spec.

### Pause TDD Contracts

19. **TDD state capture.** When pausing, the pause command MUST record:
    - Current TDD phase (RED/GREEN/REFACTOR)
    - List of currently failing tests (if any)
    - Last test run command and results
    This information is written to the Resume Context section.

### Status TDD Contracts

20. **TDD indicators in status output.** The status command MUST show:
    - Current TDD phase (RED/GREEN/REFACTOR)
    - TEST vs IMPL phase counts and completion
    - Number of TDD Log entries vs expected entries
    - Last test run results summary

## Icon Standards

All commands and SKILL.md must use these standardized icons:

**Registry-level** (active/paused/completed specs):
- `->` active
- `||` paused
- `+` completed

**Task-level** (done/in-progress/pending):
- `+` done
- `->` in-progress / current
- `o` pending

## Release Checklist

1. `claude plugin validate .claude-plugin/plugin.json` passes.
2. `claude plugin validate .claude-plugin/marketplace.json` passes without
   warnings.
3. Paths referenced in docs and templates exist (excluding placeholder paths).
4. Command contracts in this file still match `commands/*.md` and `SKILL.md`.
5. TDD contracts are enforced by all commands and the universal skill.
6. `SKILL.md` is self-contained for standalone use (spec template inlined,
   researcher fallback documented).
7. Status icons are consistent across all files (see Icon Standards above).
