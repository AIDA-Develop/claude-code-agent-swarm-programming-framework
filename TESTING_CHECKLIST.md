# Testing Checklist

> **Purpose:** This checklist is used by the **Test Agent** to ensure all code produced by the swarm is thoroughly, meaningfully tested before delivery to the user.
>
> **Workflow:** Identify testable code → Write tests per this checklist → Measure coverage → Flag untestable code for review → Attach results to `FINAL_OUTPUT_TEMPLATE.md`.

---

## Testing Philosophy

1. **Tests prove the code works**, not that lines are visited.
2. **Every test must fail** if the code it tests is broken. A test that never fails is worthless.
3. **Prefer clarity over cleverness** in test code. Tests are documentation.
4. **Test behavior, not implementation.** Refactoring without behavior changes should not break tests.
5. **If it can't be tested easily, the design may be wrong.**

---

## 1. Test Categories

### 1.1 Happy Path Tests

> Verify that the code works correctly with valid, expected input.

| # | Check Item | Status |
|---|-----------|--------|
| 1.1.1 | Primary functionality works with typical input | [ ] |
| 1.1.2 | Default parameters/options produce correct output | [ ] |
| 1.1.3 | Common usage patterns are covered | [ ] |
| 1.1.4 | Success return values are correct type and content | [ ] |

### 1.2 Edge Case Tests

> Verify behavior at boundaries and unusual but valid inputs.

| # | Check Item | Status |
|---|-----------|--------|
| 1.2.1 | Empty input (empty string, empty array, zero, None) | [ ] |
| 1.2.2 | Single-element input (smallest non-empty case) | [ ] |
| 1.2.3 | Maximum-size input (within limits) | [ ] |
| 1.2.4 | Boundary values (e.g., max integer, string length limits) | [ ] |
| 1.2.5 | Input with all identical elements | [ ] |
| 1.2.6 | Input at exact capacity thresholds | [ ] |
| 1.2.7 | Unicode / special characters in string inputs | [ ] |
| 1.2.8 | Whitespace-only or whitespace-surrounded input | [ ] |

### 1.3 Invalid Input Tests

> Verify that the code rejects or handles invalid input appropriately.

| # | Check Item | Status |
|---|-----------|--------|
| 1.3.1 | Null / None / nil input where not expected | [ ] |
| 1.3.2 | Wrong type input (if dynamically typed) | [ ] |
| 1.3.3 | Out-of-range numeric values (negative, overflow) | [ ] |
| 1.3.4 | Malformed structured data (invalid JSON, bad headers) | [ ] |
| 1.3.5 | Invalid file paths or URLs | [ ] |
| 1.3.6 | Permission-denied scenarios (file system, network) | [ ] |

### 1.4 Error Handling Tests

> Verify that error conditions are handled correctly and gracefully.

| # | Check Item | Status |
|---|-----------|--------|
| 1.4.1 | Error type/exception is correct for the failure mode | [ ] |
| 1.4.2 | Error message is descriptive and helpful | [ ] |
| 1.4.3 | Error contains appropriate context (not swallowed) | [ ] |
| 1.4.4 | Recovery / cleanup happens on error paths | [ ] |
| 1.4.5 | No panic / crash on expected failure modes | [ ] |
| 1.4.6 | Timeout / retry behavior is correct (if applicable) | [ ] |

### 1.5 Regression Tests

> Add a test for every bug found to prevent re-introduction.

| # | Check Item | Status |
|---|-----------|--------|
| 1.5.1 | Each fixed bug has a regression test | [ ] |
| 1.5.2 | Regression test fails with the old (buggy) code | [ ] |
| 1.5.3 | Regression test is documented with issue/description comment | [ ] |

### 1.6 Performance Tests (When Relevant)

> Verify performance characteristics for performance-sensitive code.

| # | Check Item | Status |
|---|-----------|--------|
| 1.6.1 | Benchmark establishes baseline for hot paths | [ ] |
| 1.6.2 | Benchmark runs as part of CI (or on-demand target) | [ ] |
| 1.6.3 | Memory usage is bounded and tested (if critical) | [ ] |
| 1.6.4 | Performance does not degrade by >10% vs. baseline | [ ] |

---

## 2. Test Structure

### 2.1 Naming Conventions

| # | Check Item | Status |
|---|-----------|--------|
| 2.1.1 | Test names describe the scenario and expected outcome | [ ] |
| 2.1.2 | Naming convention is consistent across all tests | [ ] |
| 2.1.3 | Test modules/files are named to match the code under test | [ ] |

**Naming Patterns:**

| Pattern | Example |
|---------|---------|
| `test_<function>_<scenario>_<expected>` | `test_parse_date_valid_string_returns_ok` |
| `test_<function>_given_<condition>_then_<result>` | `test_calculate_given_negative_input_then_returns_error` |
| `should_<expected>_when_<scenario>` | `should_return_error_when_file_not_found` |
| `it_<action>_<condition>` | `it_parses_valid_json_correctly` |

### 2.2 Arrange-Act-Assert Pattern

Every test should follow the AAA pattern:

```
[ARRANGE] Set up the test data, mocks, and preconditions
[ACT]     Execute the function/method under test
[ASSERT]  Verify the outcome matches expectations
```

| # | Check Item | Status |
|---|-----------|--------|
| 2.2.1 | Arrange section is clearly identifiable | [ ] |
| 2.2.2 | Act section calls exactly one unit under test | [ ] |
| 2.2.3 | Assert section checks exactly the expected outcome | [ ] |
| 2.2.4 | Blank lines or comments separate the three sections | [ ] |

### 2.3 One Assertion Per Test (When Practical)

| # | Check Item | Status |
|---|-----------|--------|
| 2.3.1 | Each test verifies a single concept or scenario | [ ] |
| 2.3.2 | Multiple related assertions on the same object are acceptable | [ ] |
| 2.3.3 | No "catch-all" tests that verify 10 unrelated things | [ ] |

> **Exception:** Checking multiple properties of a single returned object (e.g., `user.name == "Alice"` AND `user.age == 30`) is fine. Checking `add(1,1) == 2` AND `multiply(2,3) == 6` in the same test is not.

### 2.4 Test Independence

| # | Check Item | Status |
|---|-----------|--------|
| 2.4.1 | Tests do not share mutable state | [ ] |
| 2.4.2 | Tests can run in any order | [ ] |
| 2.4.3 | Tests clean up after themselves (temp files, global state) | [ ] |
| 2.4.4 | Tests do not depend on external services (mocked instead) | [ ] |
| 2.4.5 | Tests are deterministic (same input → same output) | [ ] |

---

## 3. Coverage Requirements

### 3.1 Core Logic Coverage

| # | Requirement | Status |
|---|------------|--------|
| 3.1.1 | Every public function/method has at least one test | [ ] |
| 3.1.2 | Every conditional branch (`if`, `match`, `switch`) is tested | [ ] |
| 3.1.3 | Every significant private/helper function is tested | [ ] |
| 3.1.4 | Complex algorithms have multiple test cases | [ ] |

### 3.2 Error Path Coverage

| # | Requirement | Status |
|---|------------|--------|
| 3.2.1 | Every error return path is tested | [ ] |
| 3.2.2 | Every `Result::Err` or exception path is tested | [ ] |
| 3.2.3 | Every early return (guard clause) is tested | [ ] |

### 3.3 Boundary Condition Coverage

| # | Requirement | Status |
|---|------------|--------|
| 3.3.1 | Min/max values for numeric inputs are tested | [ ] |
| 3.3.2 | Empty collections are tested | [ ] |
| 3.3.3 | Collection size exactly at capacity limit is tested | [ ] |
| 3.3.4 | String length boundaries are tested | [ ] |
| 3.3.5 | Time/duration boundaries are tested (if applicable) | [ ] |

### Coverage Targets

| Code Type | Minimum Coverage |
|-----------|-----------------|
| Core business logic | 90%+ |
| Utility functions | 80%+ |
| Error handling paths | 100% |
| Configuration / boilerplate | 50%+ (or justified) |
| Generated code | Exempt (documented) |

---

## 4. Per-Language Testing Guide

### 4.1 Rust Testing

| # | Check Item | Status |
|---|-----------|--------|
| 4.1.1 | Unit tests are in `#[cfg(test)]` modules or `tests/` directory | [ ] |
| 4.1.2 | Doc tests (`/// ````) exist for public API examples | [ ] |
| 4.1.3 | Integration tests are in `tests/` directory as separate files | [ ] |
| 4.1.4 | `cargo test` passes with no warnings | [ ] |
| 4.1.5 | `#[should_panic]` tests are used for unrecoverable error paths | [ ] |
| 4.1.6 | `rstest` or parameterized tests are used for multiple similar cases | [ ] |
| 4.1.7 | Mocking uses `mockall` or manual trait implementations | [ ] |
| 4.1.8 | No `.unwrap()` / `.expect()` in production paths (test paths OK) | [ ] |

**Rust Test Example:**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn parse_config_valid_toml_returns_settings() {
        // Arrange
        let input = r#"timeout = 30
port = 8080"#;

        // Act
        let result = Config::parse(input);

        // Assert
        assert!(result.is_ok());
        let config = result.unwrap();
        assert_eq!(config.timeout, 30);
        assert_eq!(config.port, 8080);
    }

    #[test]
    fn parse_config_invalid_syntax_returns_error() {
        // Arrange
        let input = "not valid toml {{{";

        // Act
        let result = Config::parse(input);

        // Assert
        assert!(result.is_err());
    }

    #[test]
    #[should_panic(expected = "timeout must be positive")]
    fn new_with_zero_timeout_panics() {
        Config::new(0, 8080);
    }
}
```

### 4.2 TypeScript Testing (Vitest / Jest)

| # | Check Item | Status |
|---|-----------|--------|
| 4.2.1 | Tests use `describe` blocks to group related tests | [ ] |
| 4.2.2 | `beforeEach` / `afterEach` used for setup/teardown | [ ] |
| 4.2.3 | Mocking uses `vi.fn()` (Vitest) or `jest.fn()` appropriately | [ ] |
| 4.2.4 | Async tests use `async/await` (not done callbacks) | [ ] |
| 4.2.5 | Typed mocks preserve type safety | [ ] |
| 4.2.6 | `npm test` / `vitest run` passes cleanly | [ ] |
| 4.2.7 | Snapshot tests are used only for stable outputs (not IDs, dates) | [ ] |

**TypeScript Test Example:**
```typescript
import { describe, it, expect, vi } from 'vitest';
import { UserService } from './user-service';

describe('UserService', () => {
  describe('findById', () => {
    it('returns user when found', async () => {
      // Arrange
      const mockRepo = { findById: vi.fn().mockResolvedValue({ id: '1', name: 'Alice' }) };
      const service = new UserService(mockRepo);

      // Act
      const user = await service.findById('1');

      // Assert
      expect(user).toEqual({ id: '1', name: 'Alice' });
      expect(mockRepo.findById).toHaveBeenCalledWith('1');
    });

    it('returns null when user not found', async () => {
      // Arrange
      const mockRepo = { findById: vi.fn().mockResolvedValue(null) };
      const service = new UserService(mockRepo);

      // Act
      const user = await service.findById('999');

      // Assert
      expect(user).toBeNull();
    });
  });
});
```

### 4.3 Julia Testing (Test.jl)

| # | Check Item | Status |
|---|-----------|--------|
| 4.3.1 | Tests are in `test/runtests.jl` with `@testset` blocks | [ ] |
| 4.3.2 | `@testset` names describe the feature being tested | [ ] |
| 4.3.3 | `@test` macro used for assertions | [ ] |
| 4.3.4 | `@test_throws` used for exception testing | [ ] |
| 4.3.5 | `@testset` blocks are nested for organization | [ ] |
| 4.3.6 | Custom `TestSet` types used for specialized reporting (if needed) | [ ] |
| 4.3.7 | `Pkg.test()` passes cleanly | [ ] |

**Julia Test Example:**
```julia
using Test
using MyPackage

@testset "solve_equation" begin
    @testset "happy path" begin
        @test solve(2.0, -4.0, 2.0) ≈ [1.0]  # one root
        @test solve(1.0, -5.0, 6.0) ≈ [2.0, 3.0]  # two roots
    end

    @testset "edge cases" begin
        @test solve(1.0, 0.0, 0.0) ≈ [0.0]  # zero root
        @test_throws DomainError solve(1.0, 0.0, 1.0)  # no real roots
    end

    @testset "invalid input" begin
        @test_throws ArgumentError solve(0.0, 1.0, 1.0)  # not quadratic
        @test_throws MethodError solve("a", "b", "c")  # wrong types
    end
end
```

### 4.4 C++ Testing (Catch2 / GoogleTest / doctest)

| # | Check Item | Status |
|---|-----------|--------|
| 4.4.1 | `TEST_CASE` / `TEST` / `TEST_F` used for each scenario | [ ] |
| 4.4.2 | `SECTION` / `TEST_P` used for shared setup variations | [ ] |
| 4.4.3 | `REQUIRE` for fatal assertions, `CHECK` for non-fatal | [ ] |
| 4.4.4 | RAII in tests—no manual `new`/`delete` | [ ] |
| 4.4.5 | Test executable builds and runs cleanly | [ ] |
| 4.4.6 | Tests run under ASan/UBSan with no issues | [ ] |
| 4.4.7 | Fixture classes used for common setup (GoogleTest) | [ ] |

**C++ Test Example (Catch2):**
```cpp
#include <catch2/catch_test_macros.hpp>
#include <catch2/catch_approx.hpp>
#include "calculator.hpp"

TEST_CASE("Calculator::divide", "[calculator]") {
    Calculator calc;

    SECTION("dividing two positive numbers") {
        REQUIRE(calc.divide(10.0, 2.0) == Catch::Approx(5.0));
    }

    SECTION("dividing by zero throws") {
        REQUIRE_THROWS_AS(calc.divide(10.0, 0.0), std::invalid_argument);
    }

    SECTION("dividing negative numbers") {
        REQUIRE(calc.divide(-10.0, 2.0) == Catch::Approx(-5.0));
    }
}
```

---

## 5. When Tests Cannot Be Written

Sometimes code is inherently difficult or impossible to unit test. This section defines the escalation path.

### 5.1 Legitimate Reasons for Low Testability

| Reason | Example | Action |
|--------|---------|--------|
| External hardware dependency | Embedded sensor reader | Write integration test; mock the hardware interface |
| Nondeterministic external system | Random number generator with entropy pool | Mock the entropy source; test the consumer |
| OS-level interaction | Signal handlers, process forking | Integration test in CI; minimal unit logic |
| Third-party API without stubs | Proprietary closed API | Record/replay with VCR-like tools; contract test |
| Performance-critical inline assembly | SIMD intrinsics | Property-based test inputs/outputs; benchmark |

### 5.2 Illegitimate Reasons (Code Must Be Refactored)

| Reason | Problem | Solution |
|--------|---------|----------|
| "It's just I/O" | I/O and logic are mixed | Extract pure logic; test it; I/O is thin wrapper |
| "It's too complex to test" | God function with 500 lines | Decompose into testable units |
| "It uses global state" | Functions depend on mutable globals | Inject dependencies as parameters |
| "The constructor does too much" | Can't instantiate without side effects | Separate construction from initialization |

### 5.3 Flag for Review Template

When tests cannot be written, document it:

```markdown
## Untestable Code Flag

**File:** `src/network/hardware_reader.rs`
**Function:** `read_sensor_value()`
**Reason:** Direct hardware register access via `unsafe { ptr::read_volatile(...) }`
**Risk Level:** Medium
**Mitigation:**
- Hardware interface is abstracted behind `SensorTrait`
- Consumer code tests use `MockSensor`
- Integration test exists in `tests/hardware_integration.rs` (CI: manual trigger only)
**Reviewer Sign-off:** ___________
```

---

## 6. Coverage Explanation Template

After running tests, fill in this template and attach to the final output:

```markdown
## Test Coverage Report

**Test Runner Command:** `___________`
**Date:** ___________
**Overall Coverage:** ___%

### Coverage by Module

| Module | Line Coverage | Branch Coverage | Notes |
|--------|--------------|-----------------|-------|
| `src/core/parser.rs` | ___% | ___% | _______ |
| `src/core/engine.rs` | ___% | ___% | _______ |
| `src/io/file_reader.rs` | ___% | ___% | _______ |
| `src/api/handlers.rs` | ___% | ___% | _______ |

### What Is Covered

- [ ] All public API functions
- [ ] All error return paths
- [ ] All boundary conditions
- [ ] All happy paths
- [ ] All configuration permutations

### What Is NOT Covered (Justified)

| Item | Reason | Risk |
|------|--------|------|
| `unsafe` block at L45 | Hardware access; tested via integration | Low |
| `main()` function | Thin wrapper; logic is in `run()` | Low |
| `Display` trait impl | Boilerplate; trivial | None |

### Action Items

- [ ] Improve coverage in `___________` to ___%
- [ ] Add boundary test for `___________`
- [ ] Add error path test for `___________`
```

---

## Test Checklist Summary

Use this as a quick final verification before sign-off:

| Category | All Items Passed |
|----------|-----------------|
| Happy path tests | [ ] |
| Edge case tests | [ ] |
| Invalid input tests | [ ] |
| Error handling tests | [ ] |
| Test independence verified | [ ] |
| Naming conventions followed | [ ] |
| AAA pattern used | [ ] |
| Core logic coverage >= 90% | [ ] |
| Error path coverage = 100% | [ ] |
| No flaky or external-dependent tests | [ ] |
| All tests pass in CI | [ ] |
| Untestable code flagged and justified | [ ] |

**Test Agent Sign-Off:**
```
Reviewer: ___________
Date: ___________
All tests pass: [ ] Yes  [ ] No
Coverage acceptable: [ ] Yes  [ ] No (flagged items documented above)
```
