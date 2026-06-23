# Final Output Template

> **Purpose:** This template defines the exact structure Claude Code must follow when presenting completed code to the user. Every section is required. Fill in all placeholders before delivery.
>
> **Rule:** The user sees ONLY this output. All internal reasoning, scratch work, and failed attempts must be excluded.

---

## Project Summary

<!-- 1-2 sentence overview of what was built -->

[Write a concise description of the project: what it does, what problem it solves, and the key approach taken.]

**Example:** A high-performance log file processor that parses, filters, and aggregates log entries using streaming I/O. Processes multi-gigabyte files with constant memory usage using a zero-copy parser.

---

## Language Choice

**Selected Language:** [Rust / TypeScript / Julia / C++]

### Why This Language Was Chosen

<!-- Brief, specific justification tied to the project requirements -->

[Explain the primary reasons this language was selected. Reference specific project requirements—performance needs, ecosystem libraries, deployment target, development velocity, etc. Be honest and specific.]

**Example:** Rust was chosen because the project requires zero-allocation streaming of multi-gigabyte files (memory safety without GC pauses), and the `regex` crate provides compiled regex performance unavailable in garbage-collected alternatives. Cross-platform CLI deployment via `cargo build --release` is a secondary factor.

### Why Other Languages Were Not Chosen

<!-- Brief rejection reasoning for each considered alternative -->

| Language | Reason for Rejection |
|----------|---------------------|
| Rust | [If not chosen: Why not? If chosen: N/A] |
| TypeScript | [If not chosen: Why not?] |
| Julia | [If not chosen: Why not?] |
| C++ | [If not chosen: Why not?] |

**Example:**

| Language | Reason for Rejection |
|----------|---------------------|
| Rust | N/A — selected |
| TypeScript | Node.js GC pauses unacceptable for consistent streaming throughput; lacks zero-copy file processing primitives |
| Julia | JIT warm-up latency unacceptable for CLI cold-start; ecosystem focused on numerical computing, not log processing |
| C++ | Would require manual memory management and error-prone string handling; slower development velocity for equivalent safety guarantees |

---

## Architecture

<!-- High-level design description. Diagram in ASCII or text is encouraged. -->

[Describe the architecture: major components, data flow, and design decisions. Keep it high-level—implementation details go in the code.]

```
[User Input]
    |
    v
[Component A] --> [Component B] --> [Component C]
    |                |                |
    v                v                v
[Output/Result]
```

**Key Design Decisions:**

1. **[Decision Name]:** [Why this approach was chosen over alternatives.]
2. **[Decision Name]:** [Why this approach was chosen over alternatives.]
3. **[Decision Name]:** [Why this approach was chosen over alternatives.]

---

## Files Created

<!-- List every file with a one-line description -->

| File | Description |
|------|-------------|
| `Cargo.toml` / `package.json` / `Project.toml` / `CMakeLists.txt` | Project manifest and dependencies |
| `src/main.rs` / `src/index.ts` / `src/MyPkg.jl` / `src/main.cpp` | Application entry point |
| `src/[module].rs` / `src/[module].ts` / `src/[module].jl` / `src/[module].cpp` | [What this module does] |
| `tests/[test_file].rs` / `[file].test.ts` / `test/runtests.jl` / `tests/[test_file].cpp` | Test suite |
| `[config/config.toml]` | Configuration file (if applicable) |
| `[README.md]` | Project documentation (if applicable) |
| `[Makefile]` | Build automation (if applicable) |

**Total Files:** [N]
**Total Lines of Code:** [N]
**Total Lines of Tests:** [N]

---

## How to Install

<!-- Step-by-step installation instructions. Assume a clean machine. -->

### Prerequisites

- [ ] [Prerequisite 1, e.g., "Rust 1.75+ (`rustc --version`)"]
- [ ] [Prerequisite 2, e.g., "Node.js 20+ (`node --version`)"]
- [ ] [Prerequisite 3, e.g., "Julia 1.10+ (`julia --version`)"]
- [ ] [Prerequisite 4, e.g., "g++ 13+ or clang++ 16+"]

### Installation Steps

```bash
# Step 1: Clone or navigate to the project directory
cd [project-directory]

# Step 2: Install dependencies
[command, e.g., `cargo build` / `npm install` / `julia --project=. -e 'using Pkg; Pkg.instantiate()'` / `cmake -B build`]

# Step 3: Verify installation
[command, e.g., `cargo test --lib` / `npm test` / `julia --project=. -e 'using Pkg; Pkg.test()'` / `cd build && make && ctest`]
```

---

## How to Run

<!-- Step-by-step instructions to run the application -->

### Basic Usage

```bash
[command to run the application with typical arguments]
```

### Example

```bash
$ [command with example args]

[Expected output]
```

### All Available Options

```
[Show help output or list of options/flags]
```

---

## How to Test

<!-- Step-by-step instructions to run the test suite -->

### Run All Tests

```bash
[command, e.g., `cargo test` / `npm test` / `julia --project=. test/runtests.jl` / `cd build && ctest`]
```

### Run Specific Test

```bash
[command to run a specific test or subset]
```

### Run with Coverage

```bash
[command to generate coverage report, if applicable]
```

### Test Results Summary

```
[Copy the actual test output here]

test result: ok. 42 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

---

## Best Practices Review

<!-- Completed by the Best Practices Review Agent using REVIEW_CHECKLIST.md -->

**Final Score:** [N] / 10  
**Rating:** [Excellent / Good / Acceptable / Poor / Unacceptable]

### Scoring Breakdown

| Section | Score | Notes |
|---------|-------|-------|
| Language Best Practices | [N]/10 | [Brief note] |
| Security Review | [N]/10 | [Brief note] |
| Maintainability | [N]/10 | [Brief note] |
| Performance | [N]/10 | [Brief note] |
| Simplicity | [N]/10 | [Brief note] |
| Error Handling | [N]/10 | [Brief note] |
| Naming | [N]/10 | [Brief note] |
| File Structure | [N]/10 | [Brief note] |
| Dependencies | [N]/10 | [Brief note] |
| Testing Quality | [N]/10 | [Brief note] |

### Summary

[2-3 sentence summary of code quality. Highlight strengths and any areas for future improvement.]

### Required Follow-Up (if any)

- [ ] [Item that should be addressed post-delivery]
- [ ] [Item that should be addressed post-delivery]

---

## Goal Comparison

<!-- Check each acceptance criterion against what was delivered -->

| # | Acceptance Criterion | Delivered | Status |
|---|---------------------|-----------|--------|
| 1 | [Criterion from user request] | [How it was met] | [x] Met [ ] Partial [ ] Not Met |
| 2 | [Criterion from user request] | [How it was met] | [x] Met [ ] Partial [ ] Not Met |
| 3 | [Criterion from user request] | [How it was met] | [x] Met [ ] Partial [ ] Not Met |
| 4 | [Criterion from user request] | [How it was met] | [x] Met [ ] Partial [ ] Not Met |

### Scope Decisions

[Note any deliberate scope changes: features simplified, deferred, or added with justification.]

---

## Optimization Notes

<!-- Filled by the Optimization Agent. If no optimizations were needed, state that. -->

**Optimization Status:** [No optimizations needed / N optimizations applied]

| # | What Was Optimized | Before | After | Improvement | Trade-off |
|---|-------------------|--------|-------|-------------|-----------|
| 1 | [Brief description] | [e.g., O(n^2)] | [e.g., O(n log n)] | [e.g., 10x speedup] | [None / slightly more complex] |
| 2 | [Brief description] | [Before state] | [After state] | [Improvement] | [Trade-off] |

### Benchmark Results

```
[If benchmarks were run, paste results here]
```

---

## Known Limitations

<!-- Be honest. Users should know the boundaries of what was built. -->

| # | Limitation | Impact | Workaround |
|---|-----------|--------|------------|
| 1 | [e.g., "Maximum file size limited by available memory for non-streaming operations"] | [When this matters] | [How to work around it] |
| 2 | [e.g., "Single-threaded only"] | [Performance ceiling on multi-core] | [Can be parallelized with Rayon in future] |
| 3 | [e.g., "No Windows support yet"] | [Windows users cannot use] | [PR welcome; uses POSIX APIs] |
| 4 | [e.g., "Configuration via CLI only, no config file"] | [Many options become unwieldy] | [Can add config file support later] |

### Future Improvements

- [ ] [Improvement idea with brief context]
- [ ] [Improvement idea with brief context]
- [ ] [Improvement idea with brief context]

---

## Final Code

<!-- Complete, production-ready code. Every file must be present and complete. -->

### `[filename 1]`

```[language]
[Complete file contents]
```

### `[filename 2]`

```[language]
[Complete file contents]
```

### `[filename 3]`

```[language]
[Complete file contents]
```

<!-- Repeat for all files. Do not truncate. Do not use "..." or "remaining code unchanged". Every file must be complete and ready to copy-paste. -->

---

## Appendix: Review Sign-Offs

### Best Practices Review Agent

```
Reviewer: ___________
Date: ___________
Score: ___/10
Approved: [ ] Yes  [ ] No
```

### Test Agent

```
Reviewer: ___________
Date: ___________
All Tests Pass: [ ] Yes  [ ] No
Coverage Acceptable: [ ] Yes  [ ] No
Approved: [ ] Yes  [ ] No
```

### Security Review Agent

```
Reviewer: ___________
Date: ___________
Risk Level: [ ] LOW  [ ] MEDIUM  [ ] HIGH  [ ] CRITICAL
Approved: [ ] Yes  [ ] No  [ ] Conditionally
```

### Optimization Agent

```
Reviewer: ___________
Date: ___________
Optimizations: ___ applied, ___ reverted
Net Improvement: ___%
Approved: [ ] Yes  [ ] No
```

---

> **End of Final Output Template.** All sections above must be completed before delivery to the user.
