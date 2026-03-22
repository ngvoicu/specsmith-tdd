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
7. Show a detailed breakdown with phase type markers (TEST/IMPL), explicit
   task markers, and TDD state:

```
User Auth System [active, high priority] — TDD
Created: 2026-02-10 | Updated: 2026-02-11

Phase 1: Auth Middleware Tests (TEST) [completed]
  + [TEST-AUTH-01] Write auth middleware tests (RED)
  + [TEST-AUTH-02] Write token refresh tests (RED)

Phase 2: Auth Middleware Implementation (IMPL) [in-progress]
  + [IMPL-AUTH-03] Implement verifyToken (GREEN)
  -> [IMPL-AUTH-04] Implement refreshToken <- current
  o [IMPL-AUTH-05] Add rate limiting middleware

Phase 3: OAuth Tests (TEST) [pending]
  o [TEST-AUTH-06] Write Google OAuth tests
  o [TEST-AUTH-07] Write GitHub OAuth tests

Phase 4: OAuth Implementation (IMPL) [pending]
  o [IMPL-AUTH-08] Implement Google OAuth provider
  o [IMPL-AUTH-09] Implement GitHub OAuth provider

Progress: 4/9 tasks (44%)
Current: [IMPL-AUTH-04] Implement refreshToken
Current TDD Phase: GREEN (making tests pass)
Tests: 4 written, 3 passing, 1 failing
```

Icons: `+` done, `->` in-progress/current, `o` pending.

8. Also show the Resume Context section.
9. If there are research notes (research-*.md, interview-*.md) in
   `.specs/<id>/`, mention them with file count.
