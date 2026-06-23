# Best Practices Review Checklist

> **Purpose:** This checklist is used by the **Best Practices Review Agent** to evaluate all code produced by the swarm before it reaches the user. Every item must be considered; no section is optional.
>
> **Workflow:** Review → Score each section (1-10) → Record red flags → Compute final score → Attach summary to `FINAL_OUTPUT_TEMPLATE.md`.

---

## How to Use This Checklist

1. Read the code and tests in full before scoring.
2. Score each section from **1 (poor)** to **10 (excellent)**.
3. Mark each check item `[x]` if it passes, `[ ]` if it fails or is not applicable.
4. Record any **red flags** observed—they automatically lower the section score.
5. Compute the **final weighted score** using the rubric at the bottom.
6. Attach the completed summary to the final output.

---

## 1. Language Best Practices

> Validates that the code follows idiomatic patterns for the chosen language and does not fight the language's design.

| # | Check Item | Status |
|---|-----------|--------|
| 1.1 | Code follows idiomatic conventions for the target language | [ ] |
| 1.2 | Standard library types and functions are preferred over custom equivalents | [ ] |
| 1.3 | Error handling uses the language's idiomatic mechanism (Result/Option, exceptions, etc.) | [ ] |
| 1.4 | Language-specific formatting guidelines are followed | [ ] |
| 1.5 | Compiler/linter warnings are resolved (not suppressed without justification) | [ ] |
| 1.6 | Unsafe or advanced features are used only when justified | [ ] |
| 1.7 | Common anti-patterns for the language are avoided | [ ] |

### Per-Language Notes

| Language | Key Idioms to Verify |
|----------|---------------------|
| **Rust** | Ownership/borrowing respected; `?` operator for errors; no unnecessary `.clone()`; `Iterator` methods preferred over loops; `match` exhaustive |
| **TypeScript** | Strict mode enabled; proper `async/await` usage; no `any` without justification; null-safety via unions; functional patterns preferred |
| **Julia** | Multiple dispatch leveraged; type-stable functions; broadcasting (`@.`) used; no global state mutation in hot paths; proper module structure |
| **C++** | RAII for resource management; smart pointers over raw; move semantics used; const-correctness; no manual `new`/`delete` |

**Scoring Guidance:** 10 = Flawlessly idiomatic. 5 = Mix of idiomatic and non-idiomatic patterns. 1 = Fights the language at every turn.

**Red Flags (lower score to 3 or below):**
- [ ] Manual memory management in a garbage-collected language
- [ ] Rust code that compiles with `#![allow(unsafe_code)]` without justification
- [ ] TypeScript with `any` types everywhere
- [ ] C++ with raw `new`/`delete` instead of smart pointers
- [ ] Julia with heavy mutable state in performance-critical loops

---

## 2. Security Review

> Validates that the code does not introduce vulnerabilities, leak sensitive information, or create attack surfaces.

| # | Check Item | Status |
|---|-----------|--------|
| 2.1 | All user/external input is validated before processing | [ ] |
| 2.2 | No hardcoded secrets, API keys, or credentials | [ ] |
| 2.3 | No sensitive data in error messages or logs | [ ] |
| 2.4 | Dependencies have no known CVEs (verified via `cargo audit`, `npm audit`, etc.) | [ ] |
| 2.5 | Command injection / path traversal / injection vectors are prevented | [ ] |
| 2.6 | Unsafe code blocks are justified and documented | [ ] |
| 2.7 | Cryptographic operations use established libraries (no roll-your-own) | [ ] |
| 2.8 | Concurrency primitives are used safely (no data races) | [ ] |

### Per-Language Security Notes

| Language | Tool / Check |
|----------|-------------|
| **Rust** | `cargo audit` passes; `unsafe` blocks documented with `// SAFETY:` comments |
| **TypeScript** | `npm audit` passes; no `eval()` or `Function()` constructor; prototype pollution mitigated |
| **Julia** | Package sources verified; input sanitization before `run()` or file operations |
| **C++** | ASan/UBSan clean; no buffer overflows; no use-after-free; bounds checking enabled |

**Scoring Guidance:** 10 = No security concerns. 7 = Minor concerns, well-documented. 5 = One moderate issue. 1 = Hardcoded secrets or known CVEs.

**Red Flags (lower score to 2 or below):**
- [ ] Hardcoded API key, password, or secret
- [ ] `eval()` or equivalent dynamic code execution on user input
- [ ] SQL/command injection vector present
- [ ] Unsafe code without `// SAFETY:` comment
- [ ] Known CVE in dependency with no remediation plan

---

## 3. Maintainability

> Evaluates how easy the code is to understand, modify, and extend by someone other than the original author.

| # | Check Item | Status |
|---|-----------|--------|
| 3.1 | Functions/methods are short and single-purpose (under 50 lines ideal) | [ ] |
| 3.2 | Each module/file has a clear, single responsibility | [ ] |
| 3.3 | Comments explain *why*, not *what* (code explains what) | [ ] |
| 3.4 | Public APIs are documented with doc comments | [ ] |
| 3.5 | Complex logic has inline explanations or references | [ ] |
| 3.6 | No dead code, unused imports, or commented-out blocks | [ ] |
| 3.7 | Code is organized for easy unit testing | [ ] |

**Scoring Guidance:** 10 = Crystal clear, self-documenting, easy to extend. 5 = Understandable but requires effort. 1 = Unreadable, unmaintainable mess.

**Red Flags (lower score to 3 or below):**
- [ ] Functions over 100 lines with no clear separation of concerns
- [ ] No doc comments on public API
- [ ] Extensive dead code or commented-out blocks left in place
- [ ] Magic numbers/strings with no named constants
- [ ] No separation between pure logic and I/O side effects

---

## 4. Performance

> Evaluates efficiency without premature optimization. Code should not waste resources doing unnecessary work.

| # | Check Item | Status |
|---|-----------|--------|
| 4.1 | Algorithmic complexity is appropriate for the problem | [ ] |
| 4.2 | No unnecessary allocations or copies in hot paths | [ ] |
| 4.3 | I/O operations are batched where possible | [ ] |
| 4.4 | No redundant computations or repeated work | [ ] |
| 4.5 | Data structures are chosen for the access patterns | [ ] |
| 4.6 | Concurrency is used appropriately (when beneficial) | [ ] |
| 4.7 | No blocking operations in async contexts | [ ] |

**Scoring Guidance:** 10 = Efficient and well-considered. 5 = Acceptable but has known inefficiencies. 1 = Grossly inefficient (e.g., O(n^2) where O(n) suffices).

**Red Flags (lower score to 3 or below):**
- [ ] O(n^2) or worse algorithm where O(n log n) or better is standard
- [ ] Unnecessary clones/copies in a loop
- [ ] Blocking I/O inside async runtime
- [ ] Loading entire dataset into memory when streaming would work

---

## 5. Simplicity

> Detects overengineering, unnecessary abstraction, and premature generalization.

| # | Check Item | Status |
|---|-----------|--------|
| 5.1 | Solution is the simplest approach that meets requirements | [ ] |
| 5.2 | No unnecessary abstractions or indirection layers | [ ] |
| 5.3 | No premature generalization ("what if we need...") | [ ] |
| 5.4 | Design patterns are used where they help, not where they hurt | [ ] |
| 5.5 | Code reads naturally—intent is obvious | [ ] |
| 5.6 | No unused generics, traits, or interfaces | [ ] |

**Scoring Guidance:** 10 = Elegant simplicity, every line earns its place. 5 = Mild overengineering. 1 = Enterprise FizzBuzz (17 classes for a loop).

**Red Flags (lower score to 3 or below):**
- [ ] Factory-factory pattern or excessive indirection
- [ ] Generic/trait bounds that are never utilized
- [ ] Design pattern applied where a function would suffice
- [ ] Code is harder to understand than the problem it solves

---

## 6. Error Handling

> Evaluates completeness, clarity, and user-friendliness of error handling.

| # | Check Item | Status |
|---|-----------|--------|
| 6.1 | All error cases are handled (no silent failures) | [ ] |
| 6.2 | Error types/messages are descriptive and actionable | [ ] |
| 6.3 | Error propagation uses idiomatic mechanism (not ad-hoc) | [ ] |
| 6.4 | User-facing errors are friendly; internal errors are detailed | [ ] |
| 6.5 | Panics/exceptions are reserved for unrecoverable states | [ ] |
| 6.6 | Error context is preserved through the call stack | [ ] |

### Per-Language Notes

| Language | Idiomatic Error Handling |
|----------|-------------------------|
| **Rust** | `Result<T, E>` throughout; `thiserror` or `anyhow` for ergonomics; no `.unwrap()` in production paths |
| **TypeScript** | `try/catch` with typed errors; never swallow exceptions; `Result` pattern preferred in functional code |
| **Julia** | `DomainError`, `ArgumentError` for invalid inputs; explicit error returns over exceptions for expected failures |
| **C++** | Exceptions for exceptional cases; `std::optional` / `expected` (C++23) for recoverable; RAII for cleanup |

**Scoring Guidance:** 10 = Every edge case handled gracefully. 5 = Main paths covered, some edge cases missing. 1 = `.unwrap()` everywhere or empty catch blocks.

**Red Flags (lower score to 3 or below):**
- [ ] `.unwrap()` / `.expect()` on user-controlled input
- [ ] Empty `catch` blocks or swallowed exceptions
- [ ] Panic/crash on invalid user input instead of graceful error
- [ ] Generic "something went wrong" messages for all errors

---

## 7. Naming

> Evaluates clarity, consistency, and descriptiveness of all identifiers.

| # | Check Item | Status |
|---|-----------|--------|
| 7.1 | Names are descriptive and reveal intent | [ ] |
| 7.2 | Naming conventions are consistent throughout (camelCase, snake_case, etc.) | [ ] |
| 7.3 | Boolean names suggest true/false (is_, has_, should_) | [ ] |
| 7.4 | Function names describe actions (verb or verb-noun) | [ ] |
| 7.5 | Type names describe concepts (noun or noun-phrase) | [ ] |
| 7.6 | No single-letter variables except in tiny scopes (i, x in loops OK) | [ ] |
| 7.7 | Abbreviations are standard and consistent | [ ] |

### Per-Language Conventions

| Language | Convention |
|----------|-----------|
| **Rust** | `snake_case` functions/variables; `CamelCase` types/traits; `SCREAMING_SNAKE_CASE` constants; `PascalCase` modules |
| **TypeScript** | `camelCase` variables/functions; `PascalCase` types/classes/interfaces; `SCREAMING_SNAKE_CASE` constants |
| **Julia** | `snake_case` functions; `CamelCase` types/modules; `!` suffix for mutating functions |
| **C++** | `snake_case` or `camelCase` (consistent); `PascalCase` classes; `k` or `SCREAMING` for constants |

**Scoring Guidance:** 10 = Every name is immediately clear. 5 = Mostly good, some confusing names. 1 = `doThing()`, `x1`, `x2`, `flag` everywhere.

**Red Flags (lower score to 3 or below):**
- [ ] Single-letter names in non-trivial scopes
- [ ] Inconsistent casing within the same file
- [ ] Names that contradict what the code does
- [ ] Abbreviations that are not widely known

---

## 8. File Structure

> Evaluates organization, module boundaries, and how easy it is to navigate the codebase.

| # | Check Item | Status |
|---|-----------|--------|
| 8.1 | Files are organized by responsibility (not by type) | [ ] |
| 8.2 | Module boundaries are clear and well-defined | [ ] |
| 8.3 | Public API surface is minimal and intentional | [ ] |
| 8.4 | No circular dependencies between modules | [ ] |
| 8.5 | Each file fits on one screen (under 300 lines ideal) | [ ] |
| 8.6 | Entry point / main file is obvious and minimal | [ ] |
| 8.7 | Configuration is separate from code | [ ] |

### Per-Language Structure Notes

| Language | Standard Structure |
|----------|-------------------|
| **Rust** | `src/lib.rs` or `src/main.rs` entry; `src/` modules; `tests/` integration tests; `Cargo.toml` manifest |
| **TypeScript** | `src/` source; `tests/` or `*.test.ts` co-located; `package.json` manifest; `tsconfig.json` |
| **Julia** | `src/ProjectName.jl` entry; `src/` submodules; `test/runtests.jl`; `Project.toml` |
| **C++** | `src/` and `include/` split; `tests/` directory; `CMakeLists.txt` or `Makefile`; headers for interfaces |

**Scoring Guidance:** 10 = Navigate effortlessly, every file has a clear home. 5 = Functional but messy. 1 = Everything in one file or scattered randomly.

**Red Flags (lower score to 3 or below):**
- [ ] Everything in a single file over 500 lines
- [ ] Circular module dependencies
- [ ] Public API exports everything (no encapsulation)
- [ ] Tests mixed with production code in the same files

---

## 9. Dependencies

> Evaluates whether every dependency is justified, up-to-date, and not bloated.

| # | Check Item | Status |
|---|-----------|--------|
| 9.1 | Each dependency is justified (not added "just in case") | [ ] |
| 9.2 | No duplicate functionality across dependencies | [ ] |
| 9.3 | Dependencies are current (no outdated major versions) | [ ] |
| 9.4 | No known security vulnerabilities in dependencies | [ ] |
| 9.5 | Prefer standard library over external crates/packages | [ ] |
| 9.6 | Dependency tree is reasonable (no massive transitive bloat) | [ ] |
| 9.7 | Lockfile is committed (`Cargo.lock`, `package-lock.json`, etc.) | [ ] |

### Per-Language Dependency Notes

| Language | Tool | Lockfile |
|----------|------|----------|
| **Rust** | `cargo tree` to audit; prefer `std` crates | `Cargo.lock` |
| **TypeScript** | `npm ls` / `yarn why` to audit | `package-lock.json` / `yarn.lock` |
| **Julia** | `Pkg.status()` to review; prefer stdlib | `Manifest.toml` |
| **C++** | Minimize external libs; use package manager if available | N/A (vendor or Conan/vcpkg) |

**Scoring Guidance:** 10 = Lean, justified, current dependency set. 5 = Some bloat, mostly reasonable. 1 = Left-pad situation or 200 dependencies for a CLI tool.

**Red Flags (lower score to 3 or below):**
- [ ] Dependency added for a single 5-line function
- [ ] Known CVE in a dependency with no plan to update
- [ ] Massive dependency tree for trivial functionality
- [ ] No lockfile committed for a binary/application

---

## 10. Testing Quality

> Evaluates test coverage, meaningfulness, and whether tests actually validate correctness.

| # | Check Item | Status |
|---|-----------|--------|
| 10.1 | Core business logic has unit tests | [ ] |
| 10.2 | Error paths are tested | [ ] |
| 10.3 | Boundary conditions are tested | [ ] |
| 10.4 | Tests are independent (no shared mutable state) | [ ] |
| 10.5 | Tests are deterministic (no flaky behavior) | [ ] |
| 10.6 | Test names describe the scenario being tested | [ ] |
| 10.7 | Tests run automatically via CI script or make target | [ ] |
| 10.8 | Mocking is used appropriately (not over-mocked) | [ ] |

### Per-Language Testing Notes

| Language | Test Tool | Location |
|----------|-----------|----------|
| **Rust** | Built-in `cargo test`; doc tests in `///` comments; integration tests in `tests/` | `#[cfg(test)]` modules or `tests/` |
| **TypeScript** | Vitest or Jest; co-located `*.test.ts` or `tests/` | `__tests__/` or `*.test.ts` |
| **Julia** | `Test.jl`; `@testset` blocks; `test/runtests.jl` | `test/` |
| **C++** | Catch2, GoogleTest, or doctest | `tests/` |

**Scoring Guidance:** 10 = Comprehensive, meaningful tests covering all paths. 5 = Core logic tested, edge cases missing. 1 = No tests or tests that don't assert anything.

**Red Flags (lower score to 3 or below):**
- [ ] No tests for core functionality
- [ ] Tests that don't actually assert anything
- [ ] Flaky tests that fail intermittently
- [ ] Tests depend on external services or environment state
- [ ] 100% coverage but tests are meaningless (test `return 1` against `1`)

---

## Final Score Calculation

### Weighted Scoring Table

| Section | Weight | Score (1-10) | Weighted |
|---------|--------|-------------|----------|
| Language Best Practices | 10% | ___ | ___ |
| Security Review | 20% | ___ | ___ |
| Maintainability | 15% | ___ | ___ |
| Performance | 10% | ___ | ___ |
| Simplicity | 10% | ___ | ___ |
| Error Handling | 10% | ___ | ___ |
| Naming | 5% | ___ | ___ |
| File Structure | 5% | ___ | ___ |
| Dependencies | 5% | ___ | ___ |
| Testing Quality | 10% | ___ | ___ |
| **TOTAL** | **100%** | | **___ / 10** |

### Final Score Rubric

| Score | Rating | Action Required |
|-------|--------|----------------|
| **9-10** | Excellent | Approve with optional minor notes |
| **7-8** | Good | Approve with a list of non-blocking improvements |
| **5-6** | Acceptable | Approve with mandatory follow-up issues filed |
| **3-4** | Poor | Reject—must address red flags before merge |
| **1-2** | Unacceptable | Reject—fundamental rework required |

### Red Flag Auto-Lower Rules

If any of the following are present, the **final score cannot exceed 5**:
- Hardcoded secret or credential
- Known CVE in dependencies
- `.unwrap()` / `expect()` on user input
- SQL/command injection vector
- Empty catch blocks / swallowed errors

If any of the following are present, the **final score cannot exceed 3**:
- Malicious code or intentional backdoor
- Complete absence of error handling
- Complete absence of tests for critical logic
- Data race in concurrent code

---

## Review Sign-Off

```
Review Date: ___________
Reviewer Agent: ___________
Code Language: ___________
Final Score: ___ / 10
Rating: ___________

Summary:
___________________________________________________________
___________________________________________________________

Required Changes (if any):
- [ ] _____________________________________________________
- [ ] _____________________________________________________
- [ ] _____________________________________________________

Optional Improvements:
- [ ] _____________________________________________________
- [ ] _____________________________________________________

Approved for delivery: [ ] Yes  [ ] No (requires rework)
```
