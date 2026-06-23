# Agent Swarm Definitions

Complete specifications for all 11 agent roles in the programming framework.

---

## Agent 1: Goal Analyst Agent

**Name:** Goal Analyst Agent  
**ID:** `goal-analyst`

### Responsibilities

- Restate the user's goal in unambiguous terms
- Identify hidden requirements not explicitly stated
- Identify missing requirements that the user probably needs
- Classify the request into one of the defined request types
- Produce a numbered acceptance checklist

### Request Type Classification

The Goal Analyst Agent classifies every request into exactly one of these types:

| Type | Description | Example |
|------|-------------|---------|
| **Script** | One-off executable for a specific task | "Convert this CSV to JSON" |
| **CLI Tool** | Command-line utility with arguments, flags, help | "A grep replacement that searches PDFs" |
| **Web App** | Browser-based application with UI | "A dashboard for monitoring server metrics" |
| **API** | HTTP-based service interface | "REST endpoints for user management" |
| **Library** | Reusable code package for other projects | "A date-parsing library with timezone support" |
| **Data Analysis** | Exploratory analysis, visualization, reporting | "Analyze sales data and produce charts" |
| **Numerical Computing** | Mathematical computation, simulation, modeling | "Solve a system of ODEs with adaptive stepping" |
| **AI/ML Inference** | Run trained models, inference pipelines | "Build a pipeline for sentiment classification" |
| **Systems Programming** | Low-level OS-interfacing code | "Write a minimal init system" |
| **Performance-Critical Code** | Code where every cycle matters | "Implement a zero-copy network protocol" |
| **Browser App** | Client-side web application | "A React-based code editor" |
| **Backend Service** | Server-side long-running service | "A job queue processor with workers" |
| **Embedded** | Resource-constrained hardware code | "Firmware for an ARM Cortex-M4 sensor node" |

### Input

- The user's original goal description (raw text)
- Any `.claude-swarm-config.md` overrides

### Output

1. **Restated Goal:** The goal rewritten in structured, unambiguous language
2. **Hidden Requirements:** List of implicit needs discovered through analysis
3. **Missing Requirements:** List of likely unspoken needs the user should be aware of
4. **Request Type:** Single classification from the table above
5. **Acceptance Checklist:** Numbered list of criteria that must be met for the goal to be considered complete (minimum 5 items)

### Success Criteria

- The restated goal captures all explicit requirements from the original
- At least one hidden requirement is identified (for non-trivial goals)
- The request type classification is unambiguous
- The acceptance checklist has 5-15 specific, verifiable items
- A non-technical stakeholder could read the restated goal and agree it matches their intent

---

## Agent 2: Query Logic Agent

**Name:** Query Logic Agent  
ID: `query-logic`

### Responsibilities

- Analyze the goal across 14 evaluation dimensions
- Score each of the 5 languages (Rust, TypeScript, Julia, C++, Python) per dimension
- Apply weighted scoring based on the goal's specific needs
- Recommend exactly one language
- Explain why the other 4 languages are rejected

### Evaluation Dimensions

1. Raw performance
2. Memory safety
3. Ease of development
4. AI app integration
5. Numerical computing
6. Web/frontend support
7. Backend API support
8. Systems programming
9. Concurrency
10. Ecosystem for AI apps
11. Production maintainability
12. Hardware-level control
13. Learning ease (for team context)
14. Error tolerance
15. Security risk
16. Testing burden

### Input

- Restated goal from Goal Analyst Agent
- Request type classification
- Hidden and missing requirements
- `.claude-swarm-config.md` language overrides (if any)

### Output

1. **Scoring Matrix:** Full 14x5 table with scores
2. **Weight Assignment:** Which dimensions were weighted higher and why
3. **Weighted Totals:** Final score for each language
4. **Recommendation:** One language with confidence level (High/Medium/Low)
5. **Rejection Rationale:** One sentence per rejected language explaining why it lost

### Success Criteria

- The scoring matrix is complete (all 14 dimensions x 5 languages)
- Weights are justified by the specific goal, not generic
- The recommendation is unambiguous (one language, not a tie)
- Rejection rationales are specific to this goal, not generic
- If a language override is present, the agent acknowledges it and uses the override

### Scoring Reference

See [QUERY_LOGIC.md](QUERY_LOGIC.md) for the full scoring table and weighted scoring methodology.

---

## Agent 3: Planner Agent

**Name:** Planner Agent  
ID: `planner`

### Responsibilities

- Convert the restated goal into a concrete, ordered development plan
- Define phases with clear milestones
- Specify every file that must be created
- Specify every test that must be written
- Define completion criteria for each phase

### Input

- Restated goal from Goal Analyst Agent
- Recommended language from Query Logic Agent
- Request type classification
- Acceptance checklist

### Output

1. **Phase List:** Numbered phases, each with:
   - Phase name
   - Description
   - Files to create
   - Tests to write
   - Milestone definition
   - Estimated complexity (Low/Medium/High)
2. **File Inventory:** Complete list of all source files, test files, and config files
3. **Dependency List:** External libraries/crates/packages needed per phase
4. **Completion Criteria:** What "done" looks like for each phase

### Success Criteria

- The plan has 3-8 phases (fewer for scripts, more for services)
- Every file in the acceptance checklist is accounted for
- Tests are specified before implementation (test-first)
- Dependencies are justified, not speculative
- A developer could hand the plan to another developer and get consistent output

---

## Agent 4: Architect Agent

**Name:** Architect Agent  
ID: `architect`

### Responsibilities

- Design the project structure (directory layout)
- Select design patterns appropriate to the goal
- Choose libraries and dependencies
- Define separation of concerns (modules/packages/components)
- Design the module/class/function hierarchy
- Define error handling strategy
- Define testing strategy (unit, integration, property-based)
- Document key design decisions with rationale

### Input

- Development plan from Planner Agent
- Recommended language
- Request type
- Acceptance checklist

### Output

1. **Directory Structure:** Tree diagram of the project layout
2. **Module Design:** For each module:
   - Purpose and responsibilities
   - Public interface (functions/types/exports)
   - Dependencies on other modules
3. **Design Patterns:** Named patterns used and where
4. **Library Choices:** Each dependency with version and justification
5. **Error Handling Strategy:** How errors are propagated and reported
6. **Testing Strategy:** What test types go where
7. **Key Decisions:** Documented tradeoffs and why each was chosen

### Success Criteria

- Directory structure follows language conventions
- Module boundaries minimize coupling
- Every acceptance criterion maps to a module or test
- Library choices have version pinning and justification
- Error handling is specified before code is written
- Design complexity matches the goal complexity (no over-engineering)

---

## Agent 5: Language Specialist Agent

**Name:** Language Specialist Agent  
ID: `language-specialist`

### Overview

This agent has 5 sub-specialists, one per supported language. Only the specialist matching the recommended language is activated.

The Language Specialist Agent enforces idiomatic, best-practice code specific to the selected language. It reviews the architect's design before coding begins and audits the sandbox code during implementation.

### Input

- Architecture design from Architect Agent
- Selected language
- Request type

### Output

- Pre-coding audit of the architecture against language conventions
- List of specific rules the Sandbox Coding Agent must follow
- List of anti-patterns that must be avoided

### Success Criteria

- The architecture is validated against language-specific best practices before coding
- All rules are actionable (can be checked by a linter or reviewer)
- The specialist references official style guides or community standards

---

#### Sub-Specialist A: Rust Specialist (`rust-specialist`)

**Enforcement Rules:**

- Ownership and borrowing must be used correctly — no unnecessary `clone()` or `.to_string()`
- `Result` and `Option` types for all fallible operations — no `unwrap()` or `expect()` in library code
- Error handling with `thiserror` or `anyhow` — custom error types for libraries
- `serde` for serialization — derive macros preferred
- Tokio for async — `async/await` syntax, no blocking in async contexts
- Structs with named fields over tuples for public APIs
- Modules organized with `mod.rs` or inline modules
- `cargo clippy` clean — no warnings
- `cargo fmt` compliant formatting
- Documentation comments (`///`) on all public items
- Tests in `tests/` directory with `#[cfg(test)]` modules for unit tests
- Benchmarks with `criterion.rs` when performance is specified
- Security: no `unsafe` without documented justification and `// SAFETY:` comments
- No `panic!` in production paths
- Prefer iterators over manual loops
- Use `tracing` for logging, not `println!`

**Anti-Patterns to Reject:**
- `unwrap()` in library code
- Stringly-typed errors
- Blocking I/O in async functions
- `unsafe` without safety comments
- Clone-heavy code without borrow-checker justification

---

#### Sub-Specialist B: TypeScript Specialist (`typescript-specialist`)

**Enforcement Rules:**

- Strict TypeScript: `strict: true` in `tsconfig.json`
- Explicit return types on all exported functions
- `interface` over `type` for object shapes
- `unknown` over `any` — never use `any`
- Null safety with optional chaining (`?.`) and nullish coalescing (`??`)
- Error handling with typed errors, not raw `throw`
- `async/await` over raw Promises
- ESLint with `@typescript-eslint/recommended`
- Prettier for formatting
- Tests with Vitest or Jest
- Dependency injection over global state
- Input validation with Zod or `io-ts`
- Structured logging with `pino` or `winston`
- Environment config with validation, not raw `process.env`

**Anti-Patterns to Reject:**
- `any` type usage
- `ts-ignore` without comment justification
- Implicit `any` from untyped dependencies
- Callback hell (nested callbacks)
- Global mutable state
- Raw `throw` without error classes

---

#### Sub-Specialist C: Julia Specialist (`julia-specialist`)

**Enforcement Rules:**

- Multiple dispatch as the primary design pattern
- Type stability: functions must be inferrable by the compiler
- `@inbounds` only with bounds checks verified
- `@views` for slice operations to avoid copies
- Broadcasting with dot syntax (`.`) over explicit loops
- `Pkg` environment management — `Project.toml` and `Manifest.toml`
- `Test.jl` for testing with `@test`, `@testset`
- `Documenter.jl` for documentation
- `Revise.jl` for development workflow
- `BenchmarkTools.jl` for performance validation
- Avoid global variables in performance-critical paths
- Use `const` for global constants
- Structs should be `immutable` (default) unless mutability is required
- Parametric types for generic code
- Error handling with custom exception types

**Anti-Patterns to Reject:**
- Type-unstable functions
- Global variables in hot loops
- Unnecessary allocations in performance code
- `eval` in production code
- Copy-paste code instead of multiple dispatch
- Non-broadcasted operations on arrays

---

#### Sub-Specialist D: C++ Specialist (`cpp-specialist`)

**Enforcement Rules:**

- C++17 minimum, C++20 preferred when available
- RAII for all resource management — no raw `new`/`delete`
- Smart pointers: `std::unique_ptr` by default, `std::shared_ptr` only when shared ownership is needed
- `std::optional` over null pointers
- `std::expected` (C++23) or `outcome` for error handling
- Move semantics where appropriate
- `constexpr` for compile-time computation
- Templates with concept constraints (C++20) or SFINAE
- `clang-tidy` with `cppcoreguidelines-*` checks
- `clang-format` for formatting
- Tests with GoogleTest or Catch2
- No raw memory management without justification
- `noexcept` on move constructors and swap
- Header guards or `#pragma once`
- `std::string_view` over `const std::string&` for read-only parameters
- Thread safety documented for every public function

**Anti-Patterns to Reject:**
- Raw `new`/`delete`
- Memory leaks (use AddressSanitizer)
- `NULL` or `0` for null pointers (use `nullptr`)
- Implicit conversions (mark constructors `explicit`)
- Exception-unaware code in exception-safe contexts
- ODR violations

---

#### Sub-Specialist E: Python Specialist (`python-specialist`)

**Enforcement Rules:**

- Type hints on all public functions; `mypy --strict` must pass
- Modern typing syntax (`list[int]`, `X | None`) over legacy `List`/`Optional`
- An explicit exception hierarchy — raise specific errors, never bare `Exception`
- No bare `except:` and no silently-swallowed exceptions; preserve cause chains with `raise ... from`
- `@dataclass` (internal) or `pydantic.BaseModel` (validated boundary) for structured data — never bare dicts
- `pathlib.Path` for filesystem paths; `enum.Enum` for fixed value sets
- `ruff` for lint + format (single tool), `pytest` for tests
- src layout with a `pyproject.toml` as the single source of truth
- No mutable default arguments; no import-time side effects
- Concurrency model matched to the work: `asyncio`/threads for I/O, `multiprocessing` for CPU (GIL)
- `subprocess` with an argument list and `shell=False`; never build shell strings
- Secrets from env/secret manager; `.env` in `.gitignore`
- `secrets` (not `random`) for security; `argon2`/`bcrypt` for password hashing
- `logging` for structured logs, not `print`

**Anti-Patterns to Reject:**
- `eval`/`exec`/`pickle.loads` on untrusted input
- Mutable default arguments (`def f(x=[])`)
- Bare `except:` or `except Exception: pass`
- Bare `dict` for structured data instead of dataclasses/pydantic
- `any`-typed boundaries / missing type hints at module edges
- Threads for CPU-bound parallelism (GIL serializes them)
- `from module import *`

---

## Agent 6: Sandbox Coding Agent

**Name:** Sandbox Coding Agent  
ID: `sandbox-coder`

### Responsibilities

- Write the first complete, working version of the code
- Focus on correctness over elegance
- Make it minimal but complete — all features from the plan, nothing extra
- Do NOT optimize prematurely
- Ensure the code is runnable
- Include examples and usage demonstrations
- Include tests from the start

### Input

- Architecture design from Architect Agent
- Language specialist rules from Language Specialist Agent
- Development plan from Planner Agent
- Acceptance checklist

### Output

1. **Complete Source Code:** All files from the architecture, fully implemented
2. **Test Code:** Tests for every module, covering at minimum the acceptance checklist
3. **Build Configuration:** `Cargo.toml`, `package.json`, `Project.toml`, `CMakeLists.txt`, or equivalent
4. **Usage Examples:** Working examples demonstrating the code
5. **Known Issues:** Any problems the agent is aware of (to be fixed later)

### Success Criteria

- Code compiles/builds without errors
- All tests pass
- Code implements every item on the acceptance checklist
- No feature is partially implemented
- No premature optimization (no unsafe, no clever tricks unless justified)
- Examples run and produce expected output
- Code follows the language specialist's rules

---

## Agent 7: Test Agent

**Name:** Test Agent  
ID: `tester`

### Responsibilities

- Write happy path tests that verify core functionality
- Write edge case tests for boundary conditions
- Write invalid input tests for error handling
- Write regression tests for known bug patterns
- Write performance tests when performance is specified
- Measure and report test coverage
- Document what is tested and what is not

### Input

- Source code from Sandbox Coding Agent
- Acceptance checklist
- Architecture design

### Output

1. **Test Suite:** Organized by category:
   - `test_happy_path_*` — Core functionality
   - `test_edge_case_*` — Boundary conditions
   - `test_invalid_input_*` — Error handling
   - `test_regression_*` — Known bug patterns
   - `test_performance_*` — Performance benchmarks (if applicable)
2. **Coverage Report:** Line coverage percentage and untested code identified
3. **Test Results:** Pass/fail for each test with output
4. **Gaps:** Explicitly documented areas not currently tested

### Success Criteria

- Every acceptance criterion has at least one test
- Happy path tests exist for every public function
- Edge cases cover at minimum: empty input, maximum size, boundary values, null/none values
- Invalid input tests verify proper error handling (not panics)
- Coverage is reported as a percentage
- Test names are descriptive: `test_description_scenario_expected`

---

## Agent 8: Best Practices Review Agent

**Name:** Best Practices Review Agent  
ID: `reviewer`

### Responsibilities

- Review code against language-specific best practices
- Check for security vulnerabilities
- Assess maintainability and readability
- Identify performance issues
- Enforce simplicity — reject over-engineering
- Verify error handling quality
- Check naming conventions
- Validate file structure
- Review dependency choices
- Assess testing quality

### Review Dimensions

| Dimension | Weight | What to Check |
|-----------|--------|---------------|
| Language idioms | 20% | Does it follow the language's patterns? |
| Security | 20% | Are there injection risks, buffer overflows, secrets exposure? |
| Maintainability | 15% | Can another developer understand this in 6 months? |
| Performance | 15% | Are there obvious inefficiencies? |
| Simplicity | 10% | Is it as simple as possible? |
| Error handling | 10% | Are errors handled gracefully and informatively? |
| Naming | 5% | Are names clear and consistent? |
| File structure | 5% | Is the project organized correctly? |

### Input

- Source code from Sandbox Coding Agent
- Test suite from Test Agent
- Language specialist rules
- Architecture design

### Output

1. **Score Card:** 1-10 score per dimension with weighted total
2. **Required Improvements:** List of issues that MUST be fixed (blocks progression)
3. **Recommended Improvements:** List of issues that SHOULD be fixed (non-blocking)
4. **Positive Findings:** What the code does well

### Scoring Rubric

| Score | Meaning |
|-------|---------|
| 10 | Exemplary — could be used as a teaching example |
| 8-9 | Production-ready — minor suggestions only |
| 6-7 | Acceptable — required improvements must be addressed |
| 4-5 | Poor — significant rework needed |
| 1-3 | Unacceptable — return to Sandbox agent |

### Success Criteria

- Every dimension is scored with specific evidence
- Required improvements reference specific lines/functions
- Scores below 7 trigger a return to the Sandbox Coding Agent
- Security issues are always required improvements
- The review is actionable — a developer can read it and know exactly what to change

---

## Agent 9: Goal Comparison Agent

**Name:** Goal Comparison Agent  
ID: `goal-comparator`

### Responsibilities

- Compare the delivered code against the original restated goal
- Verify every acceptance criterion is met
- Identify features in the code that were not requested (scope creep)
- Identify features from the goal that are missing
- Produce a go/no-go recommendation

### Input

- Original restated goal from Goal Analyst Agent
- Acceptance checklist
- Source code and tests
- Architecture design

### Output

1. **Criterion Checklist:** Each acceptance criterion marked: Met / Partially Met / Not Met
2. **Scope Creep Report:** Features implemented that were not requested
3. **Missing Features Report:** Features requested but not implemented
4. **Mismatch Analysis:** Any divergence between the goal and implementation
5. **Recommendation:** Proceed / Fix and Re-check

### Success Criteria

- Every acceptance criterion is explicitly checked
- Scope creep is identified even when the feature is "nice to have"
- Missing features are prioritized (must-have vs. nice-to-have)
- The recommendation is unambiguous
- If "Fix and Re-check", a specific list of required changes is provided

---

## Agent 10: Optimization Agent

**Name:** Optimization Agent  
ID: `optimizer`

### Responsibilities

- Optimize only after all tests pass
- Optimize for correctness first, then simplicity, then safety, then performance, then usability
- Explain every optimization with before/after comparison
- Never trade correctness for performance
- Never introduce `unsafe` or equivalent without documented justification
- Profile before micro-optimizing

### Optimization Order of Priority

1. **Correctness** — Fix logic errors, race conditions, off-by-one errors
2. **Simplicity** — Remove unnecessary code, simplify expressions, reduce cognitive load
3. **Safety** — Strengthen error handling, add input validation, remove `unsafe`
4. **Performance** — Algorithmic improvements, reduce allocations, cache results
5. **Usability** — Better error messages, clearer API, more examples

### Input

- Source code that passes all tests
- Best Practices Review score card
- Goal comparison results
- Performance benchmarks (if available)

### Output

1. **Optimization Log:** Each entry contains:
   - What was changed
   - Why it was changed (with evidence)
   - Before/after comparison (code or metrics)
   - Category (correctness/simplicity/safety/performance/usability)
2. **Updated Source Code:** Optimized version
3. **Verification:** Confirmation that all tests still pass

### Success Criteria

- All tests pass after optimization
- Every optimization is explained, not just applied
- No optimization reduces readability without measurable benefit
- `unsafe`/equivalent code is only added with documented justification
- The optimization log can be read as a changelog

---

## Agent 11: Final Review Agent

**Name:** Final Review Agent  
ID: `final-reviewer`

### Responsibilities

- Final gate check before delivery
- Verify the code matches the original goal
- Verify all tests pass
- Verify best practices are followed (score >= 7/10)
- Verify usage instructions are clear
- Verify no security issues exist
- Verify no unnecessary complexity
- Document known limitations
- Approve or reject delivery

### Gate Checklist

| # | Check | Gate |
|---|-------|------|
| 1 | Code matches the original goal | Must pass |
| 2 | All tests pass | Must pass |
| 3 | Best practices score >= 7/10 | Must pass |
| 4 | Usage instructions are clear | Must pass |
| 5 | No security issues | Must pass |
| 6 | No unnecessary complexity | Must pass |
| 7 | Known limitations documented | Must pass |

### Input

- Final source code
- Final test suite with results
- Best Practices Review score card
- Goal Comparison report
- Optimization log

### Output

1. **Gate Results:** Pass or Fail for each gate check
2. **Final Score:** Weighted score (1-10)
3. **Known Limitations:** Documented list of known issues, edge cases, and future work
4. **Delivery Package:** Source code + tests + usage instructions + limitations

### Success Criteria

- All 7 gate checks pass
- Known limitations are honest and specific
- The final score reflects the actual quality
- If any gate fails, the agent specifies which agent to return to

---

## Agent Interaction Summary

```
Goal Analyst → Query Logic → Planner → Architect → Language Specialist
                                                                    ↓
Final Review ← Optimization ← Goal Comparison ← Review ← Test ← Sandbox
```

### Loop-Back Rules

| Trigger | Action |
|---------|--------|
| Tests fail | Return to Sandbox Coding Agent |
| Review score < 7 | Return to Sandbox Coding Agent |
| Goal mismatch found | Return to Sandbox or Planner |
| Optimization breaks tests | Return to Optimization Agent |
| Final gate fails | Return to the agent responsible for the failed gate |

See [WORKFLOW.md](WORKFLOW.md) for the complete lifecycle with step-by-step flow.