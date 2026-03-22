---
description: Resume the active spec — read progress, load TDD context, pick up where you left off
disable-model-invocation: true
---

# Resume Spec (TDD)

Follow the "Resuming" workflow from the specsmith-tdd skill (SKILL.md).
See also `references/spec-format.md` for the SPEC.md template.

1. Read `.specs/registry.md` to find the spec with `active` status
2. If none is active, show the user their specs so they can choose one
3. Load `.specs/<id>/SPEC.md`
4. Parse progress — count completed vs total tasks per phase
5. Find the current phase and current task (`<- current` marker)
6. Read the Resume Context section
7. Parse the TDD Log to determine current TDD state:
   - If the last completed task was a TEST task with a Red entry but no
     corresponding Green entry for its IMPL task: TDD phase is RED
     (tests written and failing, ready for implementation)
   - If the last completed task was an IMPL task with a Green entry:
     TDD phase is GREEN or REFACTOR (implementation done, may need refactor)
   - If no TDD Log entries exist: starting fresh
8. Identify failing tests — scan TDD Log for TEST tasks that have Red entries
   but whose corresponding IMPL tasks are not yet checked off
9. Find the last test run command and result from TDD Log or Resume Context
10. Check if there are research notes (research-*.md, interview-*.md) in
    `.specs/<id>/` that provide additional context
11. Present a compact summary:

```
Resuming: <Title> (<id>)
Progress: <done>/<total> tasks
Phase: <phase name> (<TEST or IMPL>)
Current: <task text>
TDD Phase: RED | GREEN | REFACTOR
Failing Tests: <count and file names of tests waiting for implementation>
Last Test Run: <command, pass/fail, count>
Context: <resume context excerpt — 2-3 lines>
```

12. Begin working on the current task using the TDD workflow:
    - If current task is a TEST task: write failing tests (RED)
    - If current task is an IMPL task: make failing tests pass (GREEN)

If there are no specs at all, suggest running `/specsmith-tdd:forge` to
create one.
