---
description: Pause the active spec — save detailed resume context including TDD state for next session
disable-model-invocation: true
---

# Pause Spec (TDD)

Follow the "Pausing" workflow from the specsmith-tdd skill (SKILL.md).
See also `references/spec-format.md` for the SPEC.md template.

1. Read `.specs/registry.md` to find the spec with `active` status
2. If no active spec exists, report that there is nothing to pause and stop
3. Load the SPEC.md
4. Capture everything about the current state:
   - Which task was in progress
   - What files were modified (specific paths, function names, line ranges)
   - Key decisions made this session
   - Blockers or open questions
   - The exact next step someone should take
5. Capture TDD-specific state:
   - Current TDD phase: RED (tests written, failing), GREEN (implementation
     done, tests passing), or REFACTOR (cleaning up after green)
   - Tests written but not yet satisfied — list the test file paths and
     the test names that are currently failing, waiting for implementation
   - Last test run output: the exact command used, exit code, summary of
     results (N passed, M failed, K skipped)
   - Test files created this session — list every test file written during
     this session so the next session knows what's new
6. Write all of this to the Resume Context section in SPEC.md
7. Update checkboxes to reflect actual progress
8. Move `<- current` to the correct task
9. Add session decisions to Decision Log
10. Update TDD Log with any pending entries from this session
11. Set status to `paused`, update the `updated` date
12. Update `.specs/registry.md`
13. Confirm to the user that context was saved, including TDD state summary

Be extremely specific in the Resume Context. Write it as if briefing a
colleague who has never seen this code and will pick up tomorrow. Include
the TDD phase so they know whether to write tests or implementation next.
