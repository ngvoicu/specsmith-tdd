---
description: Show detailed progress of the active spec with TDD indicators
disable-model-invocation: true
---

# Spec Status (TDD)

Show detailed progress of the active spec with TDD state.

1. If `.specs/registry.md` does not exist, report "No specs yet" and suggest
   running `/specsmith-tdd:forge`.
2. Read `.specs/registry.md` and find the spec with `active` status.
3. If no active spec exists:
   - If specs exist, list them and ask which one to activate.
   - If no specs exist, suggest running `/specsmith-tdd:forge`.
4. Load `.specs/<id>/SPEC.md` for the active spec and parse all phases
   and tasks.
5. Parse the TDD Log to determine per-task TDD state:
   - A TEST task with a Red entry: (RED) — tests written and failing
   - An IMPL task with a Green entry: (GREEN) — tests passing
   - An IMPL task with a Green + Refactor entry: (GREEN, refactored)
   - Tasks not yet started: unmarked
6. Count aggregate test metrics from the TDD Log:
   - Tests written: count of TEST tasks checked off
   - Tests passing: count of IMPL tasks with Green entries
   - Tests failing: count of TEST tasks with Red entries whose IMPL tasks
     are not yet checked off
7. Show a detailed breakdown with task type markers (TEST/IMPL), explicit
   task markers, and TDD state:

```
User Auth System [active, high priority] — TDD
Created: 2026-02-10 | Updated: 2026-02-11

Phase 1: Auth Foundation [completed]
  + [TEST-AUTH-01] Write auth middleware tests (RED)
  + [IMPL-AUTH-02] Implement verifyToken (GREEN, refactored)
  + [TEST-AUTH-03] Write token refresh tests (RED)
  + [IMPL-AUTH-04] Implement refreshToken (GREEN)

Phase 2: OAuth Integration [in-progress]
  + [TEST-AUTH-05] Write Google OAuth tests (RED)
  + [IMPL-AUTH-06] Implement Google OAuth (GREEN)
  -> [TEST-AUTH-07] Write GitHub OAuth tests <- current
  o [IMPL-AUTH-08] Implement GitHub OAuth
  o [TEST-AUTH-09] Write token exchange tests
  o [IMPL-AUTH-10] Implement token exchange

Progress: 6/10 tasks (60%)
Current: [TEST-AUTH-07] Write GitHub OAuth tests
Last Cycle: [IMPL-AUTH-06] GREEN — 8/8 pass
TDD Phase: RED (about to write test)
Tests: 3 written, 3 passing
```

Icons: `+` done, `->` in-progress/current, `o` pending.

8. Also show the Resume Context section.
9. If there are research notes (research-*.md, interview-*.md) in
   `.specs/<id>/`, mention them with file count.
