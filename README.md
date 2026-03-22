# Spec Smith TDD

TDD-first spec management for AI coding workflows. A fork of [specsmith](https://github.com/ngvoicu/specsmith) that enforces strict test-driven development: every task starts with a failing test, no production code without red tests, and all tests are isolated.

## What It Does

- **Forge** specs with deep research including test infrastructure analysis
- **Interview** users about testing preferences (frameworks, testcontainers, coverage targets)
- Write specs with **test-interleaved phases**: test phases precede implementation phases
- **Implement** with strict **red-green-refactor**: write test → run (fail) → implement → run (pass) → refactor
- **Testcontainers** for real databases/services — no in-memory substitutes
- **Isolation**: mock at boundaries only, no external network calls in tests
- **Resume** with full TDD context: current phase (RED/GREEN/REFACTOR), failing tests, last run
- **TDD Log** audit trail proving discipline was followed

## Installation

### Claude Code Plugin
```bash
claude plugin add ngvoicu/specsmith-tdd
```

### Any AI Tool (via npx)
```bash
npx skills add ngvoicu/specsmith-tdd
```

## Commands

| Command | Description |
|---------|-------------|
| `/forge` | Research + interview + write TDD-first spec |
| `/implement` | Red-green-refactor through spec tasks |
| `/resume` | Pick up where you left off (with TDD context) |
| `/pause` | Save TDD state for next session |
| `/status` | Detailed progress with TDD indicators |
| `/switch` | Change active spec |
| `/list` | Show all specs |
| `/openapi` | Generate OpenAPI docs from codebase |

## TDD Workflow

### Forge Phase
1. Deep research (codebase + internet + test infrastructure analysis)
2. Interview with testing-specific questions
3. Write spec with Testing Architecture and test-interleaved phases

### Implementation Phase
```
TEST-AUTH-01: Write JWT tests      → RUN → FAIL (red)
TEST-AUTH-02: Write User tests     → RUN → FAIL (red)
IMPL-AUTH-03: Implement JWT        → RUN → PASS (green) → REFACTOR
IMPL-AUTH-04: Implement User       → RUN → PASS (green) → REFACTOR
```

### Spec Structure
- **Testing Architecture**: framework, isolation strategies, coverage targets
- **TEST phases**: write failing tests first
- **IMPL phases**: make tests pass with minimum code
- **TDD Log**: audit trail of red/green/refactor cycles

## Supported Languages

Works with any tech stack. Built-in knowledge for:
- **Java/Kotlin**: JUnit 5, MockK/Mockito, Testcontainers Java
- **Python**: pytest, testcontainers-python
- **TypeScript/JS**: Vitest/Jest, MSW, testcontainers-node
- **Go**: testing stdlib, testify, testcontainers-go
- **Rust**: cargo test, mockall, testcontainers-rs
- **.NET**: xUnit, Moq, Testcontainers.NET

## License

MIT
