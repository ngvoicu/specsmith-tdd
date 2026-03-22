# Spec Smith TDD

**Plan mode, but actually good — with strict test-driven development.**

Spec Smith TDD is a standalone fork of [Spec Smith](https://github.com/ngvoicu/specsmith) that enforces strict TDD in AI coding workflows. Every task starts with a failing test, no production code ships without red tests, and all tests are isolated. Specs have test-interleaved phases, a Testing Architecture section, and a TDD Log audit trail proving red-green-refactor discipline was followed.

Works with Claude Code (as a plugin), Codex, Cursor, Windsurf, Cline, Gemini CLI, and any AI coding tool that can read files.

## The Problem

Every AI coding tool has some version of "plan mode" — think before you code. But these plans are ephemeral and they don't enforce testing discipline. There's no way to:

- **Resume** a plan you were halfway through implementing
- **Switch** between multiple plans when juggling features
- **Track** which tasks are done and which are next
- **Persist** the research and decisions that informed the plan
- **Enforce TDD** — write tests first, then implement, then refactor
- **Prove discipline** — audit trail of red-green-refactor cycles

Spec Smith TDD fixes all of this.

## How It Works

### The Forge Workflow

Run `/specsmith-tdd:forge "add user authentication with OAuth"` and Spec Smith TDD takes over:

**1. Deep Research** — Exhaustive codebase scan (reads 10-20+ actual files, not just file names), web search for best practices, Context7 library docs, library comparisons, cross-skill research (frontend-design, datasmith-pg, etc.), **test infrastructure analysis** (existing frameworks, runners, mocking patterns, testcontainers, coverage tools). Everything saved to `.specs/<id>/research-01.md`.

**2. Interview** — Presents findings, states assumptions, asks targeted questions informed by the research. Not generic questions — specific ones like "I see you're using Express middleware pattern X in `src/middleware/`. Should the auth middleware follow the same pattern?" Plus **testing-specific questions**: framework preferences, isolation strategies, coverage targets, testcontainers usage. Saves answers to `interview-01.md`.

**3. Deeper Research** — Investigates the specific directions from the interview. Checks feasibility, finds edge cases, validates testing approach.

**4. More Interviews** — As many rounds as needed until every task in the spec can be described concretely. No ambiguous "figure out X" tasks.

**5. Write Spec** — Synthesizes all research and interviews into a comprehensive SPEC.md with architecture diagrams (ASCII/Mermaid), **Testing Architecture** (framework, isolation strategy, coverage targets, test commands, anti-patterns), library comparison tables, **test-interleaved phases** (TEST phase then IMPL phase), tasks with `[TEST-XX-NN]` and `[IMPL-XX-NN]` codes, **TDD Log**, a decision log, and resume context. Runs a coherence and logic review — including verifying every IMPL task references a TEST task — before presenting.

**6. Implement** — Works through the spec task by task (via `/implement`), enforcing strict red-green-refactor: write test, run it (must fail), write minimum code, run test (must pass), refactor, run test (must still pass). Every transition is logged in the TDD Log.

### Specs Are Files

Specs live in `.specs/` at your project root — plain markdown with YAML frontmatter. They diff cleanly in git, are readable in any editor, and work with any AI tool.

```
.specs/
├── registry.md                     # Denormalized index for status/progress lookups
└── user-auth-system/
    ├── SPEC.md                     # The spec document (with Testing Architecture + TDD Log)
    ├── research-01.md              # Initial codebase + web + test infra research
    ├── interview-01.md             # First interview round (incl. testing decisions)
    ├── research-02.md              # Follow-up research
    └── interview-02.md             # Second interview round
```

**SPEC.md frontmatter is authoritative.** `.specs/registry.md` is a denormalized index for quick lookups.

### A SPEC.md Looks Like This

```markdown
---
id: user-auth-system
title: User Auth System
status: active
created: 2026-02-10
updated: 2026-02-11
priority: high
tags: [auth, security, backend]
---

# User Auth System

## Overview
Add JWT-based authentication with OAuth (Google, GitHub) to the Express
API. Uses the existing middleware pattern in src/middleware/.

## Testing Architecture

### Test Framework & Tools
| Tool | Choice | Version | Purpose |
|------|--------|---------|---------|
| Test framework | Vitest | 3.0.4 | Unit and integration test runner |
| Mocking library | MSW | 2.7.0 | HTTP mocking for external APIs |
| DB testing | Testcontainers | 10.18.0 | Real PostgreSQL for integration tests |
| Coverage | v8 (Vitest) | built-in | Line and branch coverage |

### Isolation Strategy
| Layer | Approach | Services |
|-------|----------|----------|
| Domain logic | No mocks; pure functions | None |
| Service layer | Mock ports/interfaces | OAuth providers |
| Data access | Testcontainers | PostgreSQL |
| HTTP clients | MSW | Google, GitHub OAuth APIs |

### Coverage Targets
| Metric | Target |
|--------|--------|
| Line coverage | 85% |
| Branch coverage | 75% |

### Test Commands
| Command | Purpose |
|---------|---------|
| `npx vitest run` | Run all tests |
| `npx vitest --coverage` | Generate coverage report |

## Phase 1: Tests for Auth Foundation [completed] (TEST)
- [x] [TEST-AUTH-01] Write tests for JWT generation and verification
- [x] [TEST-AUTH-02] Write tests for User model CRUD
- [x] [TEST-AUTH-03] Write tests for refresh token rotation

## Phase 2: Implement Auth Foundation [completed] (IMPL)
- [x] [IMPL-AUTH-04] Set up auth middleware -> satisfies [TEST-AUTH-01]
- [x] [IMPL-AUTH-05] Create User model with Prisma -> satisfies [TEST-AUTH-02]
- [x] [IMPL-AUTH-06] Implement JWT + refresh rotation -> satisfies [TEST-AUTH-01], [TEST-AUTH-03]

## Phase 3: Tests for OAuth Integration [in-progress] (TEST)
- [x] [TEST-AUTH-07] Write tests for Google OAuth flow
- [ ] [TEST-AUTH-08] Write tests for GitHub OAuth flow <- current
- [ ] [TEST-AUTH-09] Write tests for token exchange

## Phase 4: Implement OAuth Integration [pending] (IMPL)
- [ ] [IMPL-AUTH-10] Google OAuth provider -> satisfies [TEST-AUTH-07]
- [ ] [IMPL-AUTH-11] GitHub OAuth provider -> satisfies [TEST-AUTH-08]
- [ ] [IMPL-AUTH-12] Token exchange flow -> satisfies [TEST-AUTH-09]

---

## Resume Context
> Phase 3 in progress. Google OAuth tests written and failing as expected
> (route not implemented). Currently writing GitHub OAuth callback tests.
>
> TDD Phase: RED
> Failing Tests:
>   - `should exchange code for tokens` — 404 (route not implemented)
>   - `should reject invalid state parameter` — 404
> Last Test Run: `npx vitest run tests/auth/oauth` at 14:32, 2 passed / 3 failed
>
> Next step: Write GitHub OAuth callback tests in
> `tests/auth/oauth/github.test.ts`. Use the same pattern as Google in
> `tests/auth/oauth/google.test.ts`. The GitHub OAuth app credentials
> are in `.env` as GITHUB_CLIENT_ID and GITHUB_CLIENT_SECRET.

## Decision Log
| Date | Decision | Rationale |
|------|----------|-----------|
| 2026-02-10 | JWT over sessions | Stateless, scales for microservices |
| 2026-02-10 | Refresh token rotation | Limits damage from stolen tokens |
| 2026-02-10 | Vitest over Jest | Faster; native ESM; same project uses Vite |
| 2026-02-11 | MSW over nock | Network-level intercept; works in browser and Node |

## TDD Log
| Task | Red | Green | Refactor |
|------|-----|-------|----------|
| [TEST-AUTH-01] | `Expected: valid JWT, Received: undefined` | `3 passed` (after Phase 2) | — |
| [TEST-AUTH-02] | `relation "users" does not exist` | `4 passed` (after Phase 2) | — |
| [TEST-AUTH-03] | `Expected rotated token, got same token` | `2 passed` (after Phase 2) | — |
| [IMPL-AUTH-04] | N/A | `3 passed` | Extracted middleware config to constants |
| [IMPL-AUTH-05] | N/A | `7 passed` | — |
| [IMPL-AUTH-06] | N/A | `9 passed` | Renamed `createToken` -> `issueAccessToken` |
| [TEST-AUTH-07] | `404: route not implemented` | Pending | — |

## Deviations
| Task | Spec Said | Actually Did | Why |
|------|-----------|-------------|-----|
| AUTH-04 | Use passport.js | Direct middleware | Simpler for JWT-only; avoids passport session overhead |
```

### The TDD Difference

The key difference from [Spec Smith](https://github.com/ngvoicu/specsmith) is the strict test-driven structure:

| Aspect | Spec Smith | Spec Smith TDD |
|--------|-----------|---------------|
| Phase structure | Sequential phases | **Test-interleaved**: TEST phase then IMPL phase |
| Task codes | `[AUTH-01]` | `[TEST-AUTH-01]` and `[IMPL-AUTH-02]` |
| Task linking | Independent | IMPL tasks have `-> satisfies [TEST-XX-NN]` |
| Implementation | Write code, then test | **Write test (RED), run, implement (GREEN), refactor** |
| Test execution | Optional | **Mandatory at every RED-GREEN-REFACTOR transition** |
| Testing Architecture | Testing strategy section | **Full Testing Architecture**: framework, isolation, coverage, commands, anti-patterns |
| Audit trail | None | **TDD Log** with red/green/refactor output |
| Resume context | File paths, next step | File paths, next step, **TDD phase, failing tests, last test run** |
| Research | Codebase + web | Codebase + web + **test infrastructure analysis** |
| Interviews | Feature questions | Feature questions + **testing preferences** |
| Blocking rule | None | **No production code without a failing test** |
| Test claims | Allowed | **Must run actual tests** — no "tests would pass" |

## Installation

Two ways to use Spec Smith TDD, depending on your setup.

### Path 1: Claude Code Plugin (Full — Recommended)

Everything: all 8 slash commands (`/forge`, `/implement`, `/resume`, `/pause`, `/switch`, `/list`, `/status`, `/openapi`), researcher agent (Opus-powered deep codebase + test infrastructure analysis), and SKILL.md auto-triggers.

```bash
# In Claude Code, run:
/plugin marketplace add ngvoicu/specsmith-tdd
/plugin install specsmith-tdd
```

Or manually:
```bash
git clone https://github.com/ngvoicu/specsmith-tdd.git ~/.claude/plugins/specsmith-tdd
```

After install, just run:
```
/specsmith-tdd:forge "add user authentication"
```

### Path 2: Quick Setup via npx (Any Tool)

Installs the SKILL.md into your tool's skill/instruction directory so it knows how to read, update, and resume specs from `.specs/` with full TDD enforcement.

```bash
# Claude Code (skill only — auto-triggers, no slash commands)
npx skills add ngvoicu/specsmith-tdd -a claude-code

# OpenAI Codex
npx skills add ngvoicu/specsmith-tdd -a codex

# Cursor
npx skills add ngvoicu/specsmith-tdd -a cursor

# Windsurf
npx skills add ngvoicu/specsmith-tdd -a windsurf

# Cline
npx skills add ngvoicu/specsmith-tdd -a cline

# Gemini CLI
npx skills add ngvoicu/specsmith-tdd -a gemini
```

For Claude Code, this installs SKILL.md with auto-triggers ("resume", "what was I working on", "create a spec for X", "red green refactor"). You **don't** get slash commands or the researcher agent — use Path 1 for the full plugin.

For other tools, this installs the SKILL.md which teaches the tool the full TDD spec workflow — forging with test infrastructure analysis, red-green-refactor implementation, resuming with TDD context, and cross-session continuity.

### Comparison: Plugin vs npx

| Feature | Plugin (full) | npx (any tool) |
|---------|:---:|:---:|
| `/forge` research-interview workflow | Yes | No |
| `/implement` with red-green-refactor | Yes | No |
| `/resume`, `/pause`, `/switch` commands | Yes | No |
| Researcher subagent (Opus, deep analysis + test infra) | Yes | No |
| Auto-triggers (Claude Code only) | Yes | Yes |
| Works with Codex, Cursor, Windsurf, etc. | No | Yes |
| Multi-tool `.specs/` compatibility | Yes | Yes |

## Usage

### Claude Code Plugin Flow

```
# Start a new spec with deep research + test infrastructure analysis
/specsmith-tdd:forge "add OAuth authentication"
→ Deep research (codebase + internet + Context7 + library comparison + test infra)
→ Interview rounds (targeted questions + testing preferences)
→ Writes SPEC.md with Testing Architecture, test-interleaved phases, TDD Log
→ Coherence review (incl. TEST↔IMPL cross-references) before presenting

# Implement with strict red-green-refactor
/specsmith-tdd:implement                    # Continue from current task
/specsmith-tdd:implement phase 3            # Implement all tasks in Phase 3
/specsmith-tdd:implement all phases         # Implement everything remaining

# Generate OpenAPI spec from your codebase
/specsmith-tdd:openapi
→ Scans routes, schemas, security config
→ Writes .openapi/openapi.yaml + per-endpoint docs

# Session ends — save TDD context
/specsmith-tdd:pause
→ Writes resume context (TDD phase, failing tests, last run, next step)

# New session — pick up where you left off
/specsmith-tdd:resume
→ Reads resume context, shows TDD phase, continues from exact spot

# Juggling features
/specsmith-tdd:list                    # See all specs
/specsmith-tdd:switch auth-system      # Pauses current, activates auth-system
/specsmith-tdd:status                  # Detailed progress with TDD indicators
```

### Any Tool Flow (Codex, Cursor, Windsurf, Cline, Gemini CLI)

Once configured via `npx skills add`, every tool understands the same TDD spec lifecycle. Here's the complete workflow:

**Create a spec** — Ask the tool to plan or spec out work. It creates `.specs/<id>/SPEC.md` with Testing Architecture, test-interleaved phases, TDD Log, a decision log, and resume context.

**Resume** — The tool reads `.specs/registry.md` to find the active spec, loads the SPEC.md, finds the `← current` task, reads the Resume Context section (including TDD phase and failing tests), and continues from exactly where you left off.

**Pause** — The tool captures current state into the Resume Context section: TDD phase (RED/GREEN/REFACTOR), failing test names, last test run output, which files were modified (specific paths, function names), what was completed, the exact next step. Updates checkboxes, sets status to `paused`.

**Switch** — The tool pauses the current spec (full pause), loads the target spec, sets it to `active` in the registry, and resumes it.

**List** — The tool reads `.specs/registry.md` and shows specs grouped by status (active, paused, completed).

**Complete** — The tool verifies all tasks are checked, runs the full test suite one final time, sets status to `completed` in both the SPEC.md frontmatter and the registry.

#### Tool-specific invocation examples

**Codex** (task-based prompts):
```
"create a spec for user authentication"
"resume the auth spec"
"pause and save context"
"switch to the api-refactor spec"
"show my specs"
"mark the spec as done"
```

**Cursor / Windsurf / Cline** (chat-based):
```
"plan out a caching layer"
"what was I working on?"
"save my progress and pause"
"switch to the auth spec"
"list all specs"
"complete the current spec"
```

**Gemini CLI**:
```bash
gemini "create a spec for rate limiting"
gemini "resume"
gemini "pause and save context"
gemini "switch to auth-system"
```

## The TDD Implementation Cycle

This is the core of Spec Smith TDD — what makes it different from regular Spec Smith.

### TEST Tasks (RED Phase)

```
1. Mark task ← current
2. Write test file with assertions from the spec
3. RUN tests → must FAIL (proves tests are meaningful)
4. If tests pass: STOP — feature already exists or tests are vacuous
5. Log red output in TDD Log
6. Check off task ✓
```

### IMPL Tasks (GREEN + REFACTOR Phase)

```
1. Mark task ← current
2. Write MINIMUM code to pass referenced tests
3. RUN tests → must PASS
4. If tests fail: keep implementing until green
5. Log green output in TDD Log
6. REFACTOR: clean up, remove duplication, improve naming
7. RUN tests → must STILL PASS
8. Log refactor changes in TDD Log
9. Check off task ✓
```

### Blocking Rules

- **No production code without a failing test.** If about to write implementation before tests exist, STOP and write tests first.
- **No assumed test results.** Tests MUST be run via the actual test runner (`pytest`, `vitest`, `cargo test`, `go test`, `dotnet test`, etc.) at every RED-GREEN-REFACTOR transition. Claims like "tests would pass" are not acceptable.

## Multi-Tool Support

The spec format is pure markdown. Claude Code, Codex, Cursor, Windsurf, Cline, and Gemini CLI can all work on the same `.specs/` directory.

### Setting Up Other Tools

Most tools can be set up via npx (see [Path 2](#path-2-quick-setup-via-npx-any-tool) above):

```bash
npx skills add ngvoicu/specsmith-tdd -a <tool>
```

For manual setup, see the snippet format in [SKILL.md](SKILL.md).

### Cross-Tool Sync

All tools share the same files:
- **Task codes** — `[TEST-AUTH-03]` and `[IMPL-AUTH-04]` are the same tasks everywhere
- **`← current` marker** — Every tool knows which task is next
- **Resume Context** — TDD phase, failing tests, last run, file paths, next step
- **TDD Log** — Audit trail shared across sessions and tools
- **Phase status markers** — `[pending]`, `[in-progress]`, `[completed]`, `[blocked]`

**One rule:** Don't run two tools on the same spec simultaneously. Different specs in parallel is fine.

## The Forge Workflow (Detailed)

### Phase 1: Deep Research

Not a quick scan. The researcher reads 10-20+ files, following dependency chains, checking tests, examining config. Uses every available resource: web searches for best practices, Context7 for library docs, library comparisons, cross-skill research (frontend-design, datasmith-pg, etc.).

**Test infrastructure analysis** is a dedicated research track. The researcher examines:
- Existing test frameworks, runners, and configuration
- Mocking patterns and libraries in use
- Testcontainers setup (if any)
- Coverage tooling and thresholds
- Test directory structure and naming conventions
- CI/CD test pipeline configuration

Output saved to `.specs/<id>/research-01.md`. Covers:
- Project architecture and directory structure
- Every file touching the area of change
- Tech stack versions (from lock files, not guesses)
- How similar features are currently implemented
- Library comparisons (2-3+ candidates per choice point)
- **Test infrastructure and patterns**
- Risk assessment
- UI/UX research and design references (if applicable)

### Phase 2-4: Interviews

Targeted questions based on what research found. Not generic "what do you want?" — specific questions like:

- "I see rate limiting middleware at `src/middleware/rateLimit.ts`. Should auth endpoints use the same limiter or a stricter one?"
- "The User model uses Prisma. Should OAuth tokens go in the same schema or a separate `AuthToken` model?"

Plus **testing-specific questions**:
- "The project uses pytest with testcontainers — should we follow that pattern or is there a reason to change?"
- "Mock the payment gateway at the HTTP boundary, or use a testcontainer with a sandbox endpoint?"
- "Any minimum coverage requirement for this feature?"

Multiple rounds (typically 2-5) until every task can be described concretely. Each round saved to `interview-01.md`, `interview-02.md`, etc.

### Phase 5: Write Spec

Synthesizes everything into a comprehensive SPEC.md:
- Architecture diagrams (ASCII and/or Mermaid)
- **Testing Architecture** — framework & tools table, isolation strategy per layer, coverage targets, test commands, anti-patterns to avoid
- Library comparison table with alternatives and rationale
- **Test-interleaved phases**: TEST phase for Feature A, then IMPL phase for Feature A, then TEST for Feature B, etc.
- Tasks with `[TEST-PREFIX-NN]` and `[IMPL-PREFIX-NN]` codes, with `-> satisfies` cross-references
- **TDD Log** (empty, filled during implementation)
- Decision log, resume context, deviations table

**Coherence review (mandatory before presenting):**
1. Entire spec tells a coherent story
2. Phases are in logical dependency order
3. Every task is concrete and actionable (file paths, function names)
4. Architecture diagram matches task descriptions
5. Testing Architecture covers all feature tasks
6. Library choices are consistent throughout
7. Overview accurately summarizes what phases deliver
8. No gaps — everything implementation needs is covered by a task
9. **Every IMPL phase has a preceding TEST phase**
10. **Every `[IMPL-XX-NN]` task references at least one `[TEST-XX-NN]` task**
11. **Every `[TEST-XX-NN]` task is referenced by at least one `[IMPL-XX-NN]`**

### Phase 6: Implement

Works through the spec task by task (via `/implement`), enforcing strict TDD:
- Marks tasks `← current` as they start
- **TEST tasks**: write tests, run them, confirm they FAIL, log red output
- **IMPL tasks**: write minimum code, run tests, confirm they PASS, refactor, run tests again, log green + refactor output
- Checks off `- [x]` when done
- Updates phase status markers and registry
- Logs new decisions to the Decision Log
- Logs deviations when implementation diverges from spec
- Updates Resume Context at natural pauses

## Supported Languages

Works with any tech stack. Built-in testing knowledge in [`references/testing-knowledge.md`](references/testing-knowledge.md) covering:

| Language | Test Frameworks | Mocking | Testcontainers | Coverage | Mutation Testing |
|----------|----------------|---------|----------------|----------|-----------------|
| TypeScript/JS | Vitest, Jest, Mocha | MSW, vi.mock, Sinon | testcontainers | v8, istanbul, c8 | Stryker |
| Python | pytest, unittest | pytest-mock, unittest.mock | testcontainers | coverage.py | mutmut, cosmic-ray |
| Java | JUnit 5, TestNG | Mockito, WireMock | testcontainers | JaCoCo | Pitest |
| Kotlin | JUnit 5, Kotest | MockK | testcontainers | JaCoCo, Kover | Pitest + pitest-kotlin |
| Go | testing (stdlib) | testify/mock, gomock | testcontainers-go | go test -cover | go-mutesting |
| Rust | cargo test | mockall | testcontainers | cargo-tarpaulin | cargo-mutants |
| C# | xUnit, NUnit, MSTest | Moq, NSubstitute | Testcontainers | coverlet | Stryker.NET |

Also covers: property-based testing (fast-check, Hypothesis, jqwik, proptest), isolation patterns, and common anti-patterns (in-memory DB substitutes, mocking internals, execution order dependencies, sleep-based synchronization).

## Plan Mode

Spec Smith TDD **bypasses** Claude Code's built-in plan mode. The `/forge` command IS your planning phase — deep research, interviews, spec writing with Testing Architecture and test-interleaved phases. You don't need plan mode at all.

If you happen to be in plan mode when you run `/specsmith-tdd:forge`, Spec Smith TDD asks you to exit plan mode first (Shift+Tab), then rerun `/specsmith-tdd:forge`.

## Project Structure

```
specsmith-tdd/
├── .claude-plugin/
│   ├── plugin.json                 # Plugin metadata (v1.0.0)
│   └── marketplace.json            # Marketplace registration
├── commands/
│   ├── forge.md                    # Research + test infra analysis → interview → TDD spec
│   ├── implement.md                # Strict red-green-refactor cycle
│   ├── resume.md                   # Resume with TDD context (phase, failing tests)
│   ├── pause.md                    # Pause with TDD state
│   ├── switch.md                   # Switch between specs
│   ├── list.md                     # List all specs
│   ├── status.md                   # Detailed progress with TDD indicators
│   └── openapi.md                  # Generate OpenAPI spec from codebase
├── agents/
│   └── researcher.md               # Deep research subagent (Opus) + test infra analysis
├── references/
│   ├── spec-format.md              # SPEC.md format with Testing Architecture + TDD Log
│   ├── command-contracts.md        # Behavioral contracts (20 TDD-specific)
│   └── testing-knowledge.md        # Language-agnostic testing reference (6+ languages)
├── skills/
│   └── specsmith-tdd/
│       └── SKILL.md                # → ../../SKILL.md (symlink for plugin discovery)
├── SKILL.md                        # Universal skill (works with all tools)
└── README.md
```

## Spec Format

Full specification in [`references/spec-format.md`](references/spec-format.md). Behavioral guardrails in [`references/command-contracts.md`](references/command-contracts.md).

### Frontmatter

| Field | Required | Description |
|-------|:---:|-------------|
| `id` | Yes | URL-safe slug (e.g., `user-auth-system`) |
| `title` | Yes | Human-readable name |
| `status` | Yes | `active`, `paused`, `completed`, `archived` |
| `created` | Yes | ISO date (YYYY-MM-DD) |
| `updated` | Yes | ISO date of last modification |
| `priority` | No | `high`, `medium`, `low` (default: medium) |
| `tags` | No | YAML array |

### Conventions

- **Phase markers**: `[pending]`, `[in-progress]`, `[completed]`, `[blocked]`
- **Phase types**: `(TEST)` for test-writing phases, `(IMPL)` for implementation phases
- **Task codes**: `[TEST-PREFIX-NN]` for test tasks, `[IMPL-PREFIX-NN]` for implementation tasks — unique per task, auto-incrementing across all phases
- **Satisfies references**: `[IMPL-XX-NN] <task> -> satisfies [TEST-XX-NN]` — links implementation to the tests it makes pass
- **Task checkboxes**: `- [ ] [TEST-AUTH-01]` unchecked, `- [x] [TEST-AUTH-01]` done
- **Current task**: `← current` after the task text
- **Uncertainty**: `[NEEDS CLARIFICATION]` after the task code on unclear tasks
- **Architecture Diagram**: ASCII art or Mermaid diagrams (system design, data flow, ER, state machines)
- **Testing Architecture**: Framework & tools, isolation strategy, coverage targets, test commands, anti-patterns
- **Library Choices**: Comparison table with alternatives considered and rationale
- **TDD Log**: Red/green/refactor audit trail per task
- **Resume Context**: Blockquote with TDD phase, failing tests, last test run, file paths, exact next step
- **Decision Log**: Table with date, decision, rationale
- **Deviations**: Table tracking where implementation diverged from spec

## Why Not Just Use Plan Mode?

Plan mode is a good idea with a bad implementation. It restricts Claude to read-only tools and asks for a plan. That's it. No persistence, no research depth, no interviews, no progress tracking, and certainly no TDD enforcement.

Spec Smith TDD's `/forge` command does what plan mode should do:

- **Research depth**: Reads 10-20+ files, searches the web, pulls library docs, analyzes test infrastructure. Not a quick scan.
- **Interviews**: Asks you targeted questions based on what it found — including testing preferences, isolation strategies, and coverage targets. Multiple rounds until there's no ambiguity.
- **TDD structure**: Every spec has test-interleaved phases. Tests are written before production code. Always.
- **Red-green-refactor enforcement**: The implement command runs tests at every transition. No "tests would pass" hand-waving.
- **Audit trail**: The TDD Log proves discipline was followed — red output, green output, refactor changes for every task.
- **Persistence**: Everything is saved to files. Research notes, interviews, the spec itself, the TDD Log. Nothing lives only in context.
- **Resumability**: Close the terminal, come back next week. The spec remembers exactly where you were — including which TDD phase you're in, which tests are failing, and what the last test run looked like.
- **Multi-spec**: Juggle multiple features. Switch between them with one command.

## License

MIT
