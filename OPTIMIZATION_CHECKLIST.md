# Optimization Checklist

> **Purpose:** This checklist is used by the **Optimization Agent** to guide performance improvements without sacrificing correctness, simplicity, or safety.
>
> **Workflow:** Ensure tests pass → Profile → Identify bottlenecks → Optimize per priority order → Benchmark → Verify tests still pass → Document.

---

## Optimization Order of Priority

**These priorities are absolute. Never optimize out of order.**

| Priority | Concern | Rule |
|----------|---------|------|
| **1. Correctness** | Code must produce the right answer | A fast wrong answer is worthless. Verify with tests before and after any optimization. |
| **2. Simplicity** | Code must be understandable | An optimized mess that nobody can maintain will accumulate bugs over time. |
| **3. Safety** | Code must not introduce vulnerabilities | Unsafe code for a 5% speedup is almost never worth it. |
| **4. Performance** | Code should meet performance requirements | Optimize only after profiling identifies real bottlenecks. |
| **5. Developer usability** | Code should be pleasant to work with | API ergonomics matter. Internal complexity should not leak to users. |

> **Golden Rule:** If the code is fast enough for the requirements, stop optimizing. "Fast enough" is the goal, not "as fast as possible."

---

## Pre-Optimization Requirements

**ALL of these must be true before any optimization begins:**

| # | Requirement | Status |
|---|------------|--------|
| P.1 | All tests pass (100% green) | [ ] |
| P.2 | Code is profiled and bottlenecks are identified | [ ] |
| P.3 | Performance requirements are documented (target throughput/latency) | [ ] |
| P.4 | Baseline benchmark exists and is recorded | [ ] |
| P.5 | The optimization target is a measured bottleneck (not assumed) | [ ] |

**If any requirement is not met, optimization must not proceed.**

---

## What CAN Be Optimized

These are legitimate optimization targets when identified by profiling.

### 1. Algorithmic Complexity

| # | Check Item | Status |
|---|-----------|--------|
| 1.1 | Algorithmic complexity is appropriate for the problem size | [ ] |
| 1.2 | A better algorithm is available (e.g., hash map lookup vs. linear scan) | [ ] |
| 1.3 | Data structure choice matches access patterns (random vs. sequential) | [ ] |
| 1.4 | Sorting/comparison is minimized (e.g., early exit, pre-sorting) | [ ] |

**Before/After Complexity Check:**

```
Before: O(________) via ________________
After:  O(________) via ________________
Justification: ____________________________
```

### 2. Memory Allocations

| # | Check Item | Status |
|---|-----------|--------|
| 2.1 | Heap allocations in hot loops are eliminated or pooled | [ ] |
| 2.2 | String allocations are minimized (use buffers, `String` reuse) | [ ] |
| 2.3 | Collection capacity is pre-allocated when size is known | [ ] |
| 2.4 | Stack allocation is preferred for small, short-lived objects | [ ] |
| 2.5 | Arena allocators or object pools are used for high-frequency allocation | [ ] |

### 3. Unnecessary Copies / Clones

| # | Check Item | Status |
|---|-----------|--------|
| 3.1 | References/borrows are used instead of copies where possible | [ ] |
| 3.2 | Move semantics are leveraged (not copying when moving suffices) | [ ] |
| 3.3 | Zero-copy parsing is used for large data (views, slices) | [ ] |
| 3.4 | String cloning in loops is eliminated | [ ] |

### 4. Redundant Computations

| # | Check Item | Status |
|---|-----------|--------|
| 4.1 | Loop-invariant computations are hoisted outside loops | [ ] |
| 4.2 | Common subexpressions are computed once and reused | [ ] |
| 4.3 | Memoization/caching is used for expensive repeated computations | [ ] |
| 4.4 | Short-circuit evaluation is used (avoid computing unused values) | [ ] |

### 5. I/O Batching

| # | Check Item | Status |
|---|-----------|--------|
| 5.1 | Small I/O operations are batched into larger ones | [ ] |
| 5.2 | Buffering is used for read/write operations | [ ] |
| 5.3 | Asynchronous I/O is used when parallel work is possible | [ ] |
| 5.4 | File reads are sequential (not random) when possible | [ ] |
| 5.5 | Database queries are batched (N+1 problem eliminated) | [ ] |

### 6. Cache Locality

| # | Check Item | Status |
|---|-----------|--------|
| 6.1 | Data is accessed sequentially (not random jump) when possible | [ ] |
| 6.2 | Arrays of structs are replaced with structs of arrays (SoA) if beneficial | [ ] |
| 6.3 | Hot data fields are colocated in the same cache line | [ ] |
| 6.4 | Data structures are compact (no excessive padding/indirection) | [ ] |

### 7. Concurrency Utilization

| # | Check Item | Status |
|---|-----------|--------|
| 7.1 | CPU-bound work is parallelized across available cores | [ ] |
| 7.2 | I/O-bound work uses async/concurrent processing | [ ] |
| 7.3 | Lock contention is minimized (fine-grained locks, lock-free where safe) | [ ] |
| 7.4 | Thread pool size is bounded (not spawning unbounded threads) | [ ] |
| 7.5 | No oversubscription (more threads than cores for CPU-bound work) | [ ] |

---

## What Should NOT Be Optimized

These optimizations are forbidden unless accompanied by benchmark data proving significant benefit.

| Anti-Pattern | Why Not | Exception |
|-------------|---------|-----------|
| **Readability for marginal gains** | Code is read 10x more than written | 10%+ improvement in hot path with clear comments |
| **Micro-optimizations without benchmarks** | Compilers are smart; you're probably wrong | Confirmed via profiler that this is the bottleneck |
| **Premature abstraction** | Indirection hurts inlining and cache | The abstraction enables multiple real implementations |
| **Hand-rolled assembly / SIMD** | Compiler auto-vectorizes well | Confirmed 2x+ improvement via benchmark |
| **Unsafe code for small speedup** | Risk of UB is not worth 5% | Safety invariants proven and documented |
| **Optimizing cold paths** | No user-visible benefit | Cold path is called in a loop in some scenarios |
| **Saving bytes at the cost of clarity** | Modern machines have memory | Embedded/constrained environments only |

---

## Per-Language Optimization Notes

### Rust Optimization

| # | Check Item | Status |
|---|-----------|--------|
| R.1 | Zero-cost abstractions are used (iterators, closures compile to tight loops) | [ ] |
| R.2 | Benchmarking uses `criterion` crate (statistically rigorous) | [ ] |
| R.3 | `cargo bench` baseline is recorded before changes | [ ] |
| R.4 | `--release` profile is used for benchmarks (not debug) | [ ] |
| R.5 | No unnecessary `.clone()` / `.to_string()` in hot paths | [ ] |
| R.6 | `&str` slices are used instead of `String` where possible | [ ] |
| R.7 | `Vec::with_capacity(n)` is used when final size is known | [ ] |
| R.8 | `fold` / `try_fold` used instead of manual accumulation loops | [ ] |
| R.9 | `rayon` is used for data parallelism (parallel iterators) | [ ] |
| R.10 | `#![feature(...)]` nightly features are avoided unless necessary | [ ] |
| R.11 | `[target/release]` profile tuned in `Cargo.toml` (LTO, codegen units) | [ ] |

**Rust Benchmark Example:**
```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use mycrate::parser;

fn benchmark_parse_json(c: &mut Criterion) {
    let input = r#"{"name": "test", "value": 42}"#;
    c.bench_function("parse_json_small", |b| {
        b.iter(|| parser::parse_json(black_box(input)))
    });
}

criterion_group!(benches, benchmark_parse_json);
criterion_main!(benches);
```

### TypeScript Optimization

| # | Check Item | Status |
|---|-----------|--------|
| T.1 | Async operations are parallelized with `Promise.all()` where independent | [ ] |
| T.2 | Bundle size is audited (no importing entire library for one function) | [ ] |
| T.3 | Tree-shaking is enabled and verified | [ ] |
| T.4 | Hot paths avoid `JSON.parse` / `JSON.stringify` in loops | [ ] |
| T.5 | String concatenation uses arrays/join for many pieces | [ ] |
| T.6 | `Set` / `Map` used for O(1) lookups instead of array `.find()` | [ ] |
| T.7 | `for...of` used instead of `.forEach()` (allows break, faster) | [ ] |
| T.8 | Event listeners are debounced/throttled for high-frequency events | [ ] |
| T.9 | Database queries use projection (select only needed fields) | [ ] |
| T.10 | `node --prof` or clinic.js used for server-side profiling | [ ] |

### Julia Optimization

| # | Check Item | Status |
|---|-----------|--------|
| J.1 | Functions are type-stable (no runtime type inference in hot loops) | [ ] |
| J.2 | `@code_warntype` shows no red (unstable) types in hot paths | [ ] |
| J.3 | `@.` broadcasting macro used instead of explicit loops | [ ] |
| J.4 | `@views` used for slice operations to avoid copies | [ ] |
| J.5 | `@inbounds` used only after bounds safety is verified | [ ] |
| J.6 | `@simd` used on loops that meet requirements | [ ] |
| J.7 | `@fastmath` used only where precision tradeoff is acceptable | [ ] |
| J.8 | Pre-allocated output arrays passed to mutating functions (`!`) | [ ] |
| J.9 | `@benchmark` from `BenchmarkTools.jl` used for measurements | [ ] |
| J.10 | `@profview` used to identify hot spots | [ ] |
| J.11 | `const` globals for configuration; avoid global state in functions | [ ] |

**Julia Type Stability Check:**
```julia
# Run this on every function in a hot path
@code_warntype my_function(1.0, 2.0)
# Any red text indicates type instability that will hurt performance
```

### C++ Optimization

| # | Check Item | Status |
|---|-----------|--------|
| C.1 | Move semantics are used (`std::move`, `&&` parameters) | [ ] |
| C.2 | `const&` used for large read-only parameters | [ ] |
| C.3 | Cache-friendly data layouts (SoA vs. AoS considered) | [ ] |
| C.4 | Compiler flags: `-O2` or `-O3`, `-march=native` if appropriate | [ ] |
| C.5 | LTO (Link Time Optimization) enabled for release builds | [ ] |
| C.6 | `__attribute__((hot))` / `__attribute__((cold))` used appropriately | [ ] |
| C.7 | `noexcept` used where exceptions won't escape (enables optimization) | [ ] |
| C.8 | Small String Optimization (SSO)-aware `std::string` usage | [ ] |
| C.9 | `std::string_view` used for read-only string references | [ ] |
| C.10 | Branch prediction hints (`[[likely]]` / `[[unlikely]]` in C++20) | [ ] |
| C.11 | Google Benchmark or Catch2 benchmarks used for measurements | [ ] |

---

## Optimization Justification Template

Every optimization must be documented using this template. Attach to `FINAL_OUTPUT_TEMPLATE.md`.

```markdown
## Optimization Note #[N]

**File:** `src/________`
**Function:** `________`
**Priority Triggered:** [ ] Correctness  [ ] Simplicity  [ ] Safety  [ ] Performance  [ ] Usability

### Problem
___________________________________________________________
___________________________________________________________

### Before
```
[Code or pseudocode]
```
Complexity: O(_____)
Benchmark: _____ ops/sec / _____ ms

### After
```
[Code or pseudocode]
```
Complexity: O(_____)
Benchmark: _____ ops/sec / _____ ms

### Improvement
Speedup: _____x (or _____% faster)
Memory: _____% reduction (if applicable)

### Trade-offs
- [ ] Reduced readability (mitigated by: ____________)
- [ ] Increased complexity (justified by: ____________)
- [ ] Safety implications (documented at: ____________)
- [ ] Breaking API change (migration path: ____________)
- [ ] None

### Verification
- [ ] All tests pass after optimization
- [ ] Benchmark confirms improvement
- [ ] No regression in unrelated tests
- [ ] Unsafe code (if any) has `// SAFETY:` comment

### References
- Profile data: ____________
- Benchmark results: ____________
```

---

## Optimization Workflow Summary

```
STEP 1: Verify tests pass (100% green)
    |
STEP 2: Profile the code (find real bottlenecks)
    |
STEP 3: Record baseline benchmark
    |
STEP 4: Apply ONE optimization at a time
    |
STEP 5: Verify tests still pass
    |
STEP 6: Benchmark the change
    |
STEP 7: If no improvement → REVERT (do not keep)
    |
STEP 8: Document using justification template
    |
STEP 9: Repeat from Step 4 if more bottlenecks exist
    |
STEP 10: Final review and sign-off
```

---

## Optimization Agent Sign-Off

```
Review Date: ___________
Reviewer Agent: ___________
Code Language: ___________

Optimizations Applied: ___
Optimizations Reverted: ___
Net Performance Improvement: ___%

All Tests Pass Post-Optimization: [ ] Yes  [ ] No
Benchmarks Recorded: [ ] Yes  [ ] No

Approved for delivery: [ ] Yes  [ ] No  [ ] Conditionally

Signature: ___________
```
