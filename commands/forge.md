---
description: Research deeply, interview the user, then forge a structured TDD-first spec with test phases preceding implementation phases. This is a persistent planning workflow.
disable-model-invocation: true
---

# Forge a Spec (TDD)

You are about to run the Spec Smith TDD forge workflow. This bypasses plan mode
with something far more thorough: deep research -> interview -> more research
-> more interview -> write spec -> review.

The forge workflow never produces application code. Its only outputs are
`.specs/` files: research notes, interview notes, and the SPEC.md.

Every spec produced by this workflow uses test-driven development: test phases
come before their corresponding implementation phases.

The user's request: $ARGUMENTS

## Preflight: Resolve Spec Identity

Before starting research, resolve spec identity:

1. Generate a spec ID from the user's request (lowercase, hyphenated)
2. Collision-check read-only:
   - Check `.specs/<spec-id>/SPEC.md`
   - Check whether `<spec-id>` exists in `.specs/registry.md`
3. If the ID already exists, stop and ask the user to choose one:
   - **Resume** existing spec
   - **Rename** new spec (suggest `<spec-id>-v2`)
   - **Archive** old spec then recreate
4. Use this resolved `<spec-id>` in all later phases.

## Plan Mode Check

Before starting, check if you're in plan mode (read-only).

- If in plan mode:
  - Do not run `/specsmith-tdd:forge` in plan mode
  - Ask the user to exit plan mode (Shift+Tab), then rerun `/specsmith-tdd:forge`
  - Stop here until plan mode is exited
- If NOT in plan mode:
  - Create/initialize `.specs/<spec-id>/` before the first write
  - Persist artifacts as each phase completes

## Setup (before research)

Before starting research, ensure the directory structure exists:

1. Reuse the already-resolved `<spec-id>` from Preflight.
2. Create the spec directory:
   ```
   mkdir -p .specs/<spec-id>
   ```
3. If `.specs/registry.md` doesn't exist, initialize it with the header row.

If directory creation fails because the environment is still read-only, ask
the user to exit plan mode (Shift+Tab) and rerun `/specsmith-tdd:forge`.

## Phase 1: Deep Research

This is the most important phase. Be exhaustive. You are gathering every
piece of context needed to write a spec that won't need revision mid-build.

### 1a. Codebase Research

Scan the project thoroughly. Don't just grep for keywords — understand the
architecture:

- **Project structure**: Map the directory tree, identify patterns (monorepo?
  modules? packages?)
- **Tech stack**: Read package.json/Cargo.toml/go.mod/requirements.txt etc.
  Understand what's already in use
- **Related code**: Find every file, function, component, route, model, and
  test that touches the area the user wants to change
- **Patterns**: How does the existing code handle similar things? If adding
  auth, how is the existing middleware structured? If adding a feature, what
  patterns do similar features follow?
- **Tests**: What testing frameworks are used? What's the test coverage like
  in the relevant area?
- **Test infrastructure**: Analyze the full test setup — frameworks (Jest,
  Vitest, pytest, Go testing, JUnit), Docker and testcontainers usage,
  coverage tools (Istanbul, c8, coverage.py, go cover), test runners and
  configuration, CI test pipeline, fixture and factory patterns, mocking
  libraries (msw, nock, unittest.mock, gomock), snapshot testing, test
  database setup and teardown
- **Config**: Environment variables, build config, CI/CD pipelines that
  might be affected
- **Dependencies**: What libraries are relevant? Are there version
  constraints?

Use Glob, Grep, and Read aggressively. Read actual file contents, not just
file names. Open 10-20 files if needed.

**Always spawn the `specsmith-tdd:researcher` agent** (Task tool) to run an
exhaustive parallel research pass. The researcher reads 15-30 files, runs
3+ web searches, compares library candidates, and assesses risks. Save
structured findings to `.specs/<id>/research-01.md`. Don't skip this —
thorough research is the foundation of a spec that won't need revision
mid-build.

### 1b. Context7 & Cross-Skill Research (in parallel with researcher)

While the researcher agent runs, do these yourself — they use MCP tools
that the researcher agent doesn't have access to:

- **Context7**: If available (resolve-library-id / query-docs tools), pull
  up-to-date documentation for 2-5 key libraries involved. Check API changes,
  deprecated features, and recommended patterns for the specific versions.
  Also pull docs for testing libraries relevant to the project (Vitest, Jest,
  pytest, testcontainers, Playwright, Testing Library, etc.) to ensure test
  code follows current best practices.
- **Cross-skill loading**: Load other available skills when relevant:
  - **frontend-design**: For UI-heavy features — creative, professional design
  - **datasmith-pg**: For database work — schema design, migrations, indexing
  - **webapp-testing**: Always load for TDD specs — Playwright patterns,
    testing strategy, test architecture guidance
  - **vercel-react-best-practices**: For Next.js/React optimization
  - Any other relevant skill that's available

### 1c. UI Research (if applicable)

If the project has a UI and the changes affect it:

- Take screenshots of current state if browser tools are available
- Map the component hierarchy
- Understand the routing and state management
- Research modern UI patterns for the specific use case
- Look at design references for creative, professional approaches
- Note accessibility requirements (WCAG compliance)

### 1d. Merge & Save Research

When the researcher agent completes, read its output. Merge your Context7
and cross-skill findings into the research notes. Write the combined
findings to:
```
.specs/<spec-id>/research-01.md
```

Structure it clearly:

```markdown
# Research Notes — <Title>
## Date: <today>

## Project Architecture
<what you found about the structure>

## Relevant Code
<key files, functions, patterns found>

## Tech Stack & Dependencies
<what's in use, versions>

## Test Infrastructure
<testing frameworks, test runners, configuration files, coverage tools,
Docker/testcontainers setup, mocking libraries, fixture patterns,
CI test pipeline, existing test conventions and naming patterns,
test database setup/teardown approach>

## Library Comparison
<comparison tables for any libraries evaluated, with recommended picks>

## External Research
<web findings, library docs, best practices, Context7 findings>

## UI Research (if applicable)
<screenshots, component map, design references, accessibility notes>

## Risk Assessment
<what could go wrong, security considerations, performance implications>

## Open Questions
<things you couldn't determine from research alone>
```

## Phase 2: Interview Round 1

Now present your findings and ask targeted questions. The goal is NOT to ask
generic questions — your research should inform very specific questions.

**Structure the interview like this:**

1. **Summarize what you found** (2-3 paragraphs, not a wall of text)
2. **State your assumptions** — "Based on the codebase, I'm assuming we'll
   use X pattern because that's what similar features use. Correct?"
3. **Ask specific questions** that your research couldn't answer:
   - Architecture decisions: "Should this be a new module or extend the
     existing one in src/features/?"
   - Scope boundaries: "Should this handle X edge case or is that a
     separate spec?"
   - Technical choices: "I see you're using Library A for similar things.
     Should we stick with that or is there a reason to try Library B?"
   - User-facing behavior: "What should happen when X fails?"
4. **Ask testing-specific questions** (2-3 probes):
   - "I found <framework> for testing. Do you want to stick with it or
     switch to something else? Any preferences for test runner config?"
   - "For services that depend on <database/service>, should we use
     testcontainers with real containers or mocks? Which services need
     real containers vs lightweight mocks?"
   - "What's your mocking approach preference — mock only at external
     boundaries, or also mock internal modules? Any opinions on
     msw/nock/similar for HTTP mocking?"
   - "Do you have a coverage target in mind (e.g., 80% line coverage)?
     Any interest in mutation testing (Stryker/mutmut) for critical paths?"
5. **Propose a rough approach** and ask for reactions — don't wait for the
   user to design everything

Keep it to 5-8 questions max per round (including testing questions). More
than that overwhelms.

**STOP after asking your questions and wait for the user to answer.** Do not
answer your own questions, guess answers, or proceed to deeper research or
spec writing until the user responds. The interview is a real conversation —
the user's answers determine what gets built.

**Save the interview:**
```
.specs/<spec-id>/interview-01.md
```

```markdown
# Interview Round 1 — <Title>
## Date: <today>

## Questions Asked
1. <question>
   **Answer**: <user's response>

2. <question>
   **Answer**: <user's response>

## Key Decisions
- <decision made during this interview>

## New Research Needed
- <things to look into based on answers>
```

## Phase 3: Deeper Research (informed by interview)

Based on the user's answers, do another round of research:

- Explore the specific code paths they mentioned
- Look up the libraries or patterns they chose
- Check feasibility of the approach discussed
- Find potential issues with the chosen direction
- Research the testing approach they prefer (testcontainers setup, mock
  library docs, coverage tool configuration)

Save to:
```
.specs/<spec-id>/research-02.md
```

## Phase 4: Interview Round 2+

Present your deeper findings. Ask about:

- Trade-offs you discovered
- Edge cases that emerged from the deeper research
- Implementation sequence — "I'd suggest building X first because Y depends
  on it. Does that sequence make sense?"
- Scope refinement — "This feels like it could be split into two specs.
  Want to keep it together or separate?"
- Test phase ordering — "I'd put the test phase for <component> before its
  implementation. Here's the test-implementation pairing I have in mind..."

Save each round to `interview-02.md`, `interview-03.md`, etc.

**Repeat phases 3-4 as many times as needed.** The loop ends when:

- You have enough clarity to write a spec with no ambiguous tasks
- The user says they're satisfied with the direction
- Every task in the spec can be described concretely (not "figure out X")
- The test-implementation pairing is clear for every feature

It's fine if this takes 2 rounds or 5 rounds. Don't rush it.

## Phase 5: Write the Spec

Now synthesize everything — all research notes, all interview answers, all
decisions — into a SPEC.md.

Read the skill's `references/spec-format.md` for the full template. The
spec should include:

1. **YAML frontmatter**: id, title, status (active), created, updated,
   priority, tags
2. **Overview**: 2-4 sentences capturing the goal and scope. Someone reading
   just this section should understand what's being built and why. Mention
   that this spec follows TDD methodology.
3. **Architecture Diagram**: ASCII art or Mermaid diagram showing the system
   architecture, data flow, or component relationships. Every non-trivial spec
   should have at least one diagram. Use ASCII for simple flows, Mermaid for
   complex relationships (ER diagrams, state machines, flowcharts).
4. **Library Choices**: Table comparing evaluated libraries with the selected
   pick and rationale. Include version numbers. Format:
   `| Need | Library | Version | Alternatives | Rationale |`
5. **Testing Architecture**: Dedicated section describing the test strategy:
   - Test frameworks and runners (with versions)
   - Test directory structure and naming conventions
   - Isolation strategy: which services get testcontainers, which get mocks
   - Mocking approach and libraries
   - Coverage targets and tools
   - CI integration notes
6. **Phases**: Major milestones structured as TEST-IMPL pairs. Each feature
   area gets a TEST phase followed by its IMPL phase:
   - `Phase 1: <Feature Area> Tests [pending]` — write failing tests
   - `Phase 2: <Feature Area> Implementation [pending]` — make tests pass
   - `Phase 3: <Next Feature> Tests [pending]` — write failing tests
   - `Phase 4: <Next Feature> Implementation [pending]` — make tests pass
   - ...and so on. Every IMPL phase has a preceding TEST phase.
7. **Tasks**: Two types of tasks, each with a task code `[PREFIX-NN]`:

   **TEST tasks** (in TEST phases):
   - File path where the test will be written
   - Test descriptions (what each test asserts)
   - Isolation strategy for this test (containers, mocks, in-memory)
   - Example: `- [ ] [AUTH-01] Write auth middleware tests in tests/auth/middleware.test.ts — test valid JWT, expired JWT, missing token, malformed token (mock: HTTP client, real: nothing)`

   **IMPL tasks** (in IMPL phases):
   - What production code to write
   - Which test task(s) it satisfies, using arrow notation
   - Example: `- [ ] [AUTH-02] Implement verifyToken() in src/auth/tokens.ts -> satisfies [AUTH-01]`

   Task numbering is continuous across phases (AUTH-01, AUTH-02, AUTH-03...).

8. **Testing Strategy**: Comprehensive testing plan covering unit tests,
   integration tests, e2e tests, and edge case tests. Specify frameworks,
   test file paths, and what each test covers. Every feature task should have
   a corresponding test task.
9. **TDD Log**: Empty section ready for implementation to fill in. Format:
   ```markdown
   ## TDD Log

   | Task | Red (failing) | Green (passing) | Refactor |
   |------|--------------|-----------------|----------|
   ```
10. **Resume Context**: Write the initial context as if briefing someone who
    will start implementing tomorrow. Mention the TDD workflow: start with
    the first TEST phase, write failing tests, then move to the IMPL phase.
11. **Decision Log**: Every decision from the interviews, with rationale.

**Coherence and logic review (mandatory before presenting):**

Before presenting the spec to the user, review it for coherence and logic:

1. Read through the entire spec as a whole — does it tell a coherent story?
2. Check that phases are in logical dependency order — no phase requires
   work from a later phase
3. **Verify every implementation phase has a preceding test phase** — this
   is the TDD invariant. No IMPL phase without its TEST phase first.
4. **Verify every IMPL task references at least one TEST task** via the
   `-> satisfies [TEST-XX-NN]` notation
5. **Verify every TEST task is referenced by at least one IMPL task** — no
   orphan tests that nothing implements
6. Verify every task is concrete and actionable (file paths, function names)
7. Confirm the architecture diagram matches the task descriptions
8. Check that the testing strategy covers all feature tasks
9. Verify library choices are consistent throughout (no conflicting picks)
10. Ensure the overview accurately summarizes what the phases will deliver
11. Look for gaps — is there anything the implementation would need that
    isn't covered by a task?

**Quality check before presenting:**

- Every task should be concrete ("Add verifyToken() to src/auth/tokens.ts"),
  not vague ("implement token verification")
- Phases should have clear boundaries and dependencies
- TEST phases should be self-contained — all test files and assertions
  specified, no ambiguity about what "failing" means
- IMPL phases should be minimal — just enough code to make tests pass
- The Decision Log should capture every non-obvious choice
- The Overview should be understandable without reading the interviews
- Architecture diagrams should be clear and accurate
- UI designs should be creative, sleek, and professional — not generic
- Library choices should be the best available, modern, well-maintained

Save to:
```
.specs/<spec-id>/SPEC.md
```

Update `.specs/registry.md` (set status to `active`).

**Present the spec to the user and wait for approval.** Walk through the
phases (highlighting the TEST->IMPL pairing) and ask: "Does this look right?
Want to adjust anything before we start?" Do not begin implementing until
the user explicitly says to proceed. The spec review is a gate — the user
may want to add tasks, reorder phases, change scope, or rename things.
Respect this pause.

After user approval, implementation is handled by `/specsmith-tdd:implement`.
Do not implement application code inside `/specsmith-tdd:forge`.
