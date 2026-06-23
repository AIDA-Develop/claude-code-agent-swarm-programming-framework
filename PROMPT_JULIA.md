# Julia-Specific Claude Code Prompt

Append this prompt after `PROMPT_UNIVERSAL.md` for all Julia projects. These rules supplement -- not replace -- the universal workflow.

---

## Julia Commands

Use these commands throughout the workflow:

| Command | Purpose | When to run |
|---------|---------|-------------|
| `julia --project=.` | Start Julia with project environment | All Julia sessions |
| `] activate .` | Activate project environment (Pkg REPL) | Project initialization |
| `] instantiate` | Install all declared dependencies | After Project.toml changes |
| `] add PackageName` | Add a dependency | When adding packages |
| `] test` | Run project test suite | After implementation, must pass |
| `] status` | List installed packages | Dependency review |
| `] rm PackageName` | Remove a dependency | Cleanup |
| `using BenchmarkTools; @benchmark f(x)` | Benchmark a function | Step 13 (Optimization Pass) |
| `@btime f(x)` | Quick timing with memory allocation | Quick performance check |
| `@code_warntype f(x)` | Check for type instabilities | Code review |
| `@inferred f(x)` | Verify return type inference | Tests for type stability |
| `julia --check-bounds=yes` | Run with bounds checking enabled | Test execution |
| `julia -t auto` | Run with all available threads | Parallel execution |

---

## Julia Code Rules

### Type Stability
- Write type-stable code in all performance-critical paths.
- Use `@code_warntype f(args...)` to check for type instabilities (red `Any` or `Union{}` in output).
- Use `@inferred f(args...)` in tests to enforce type-stable return values.
- Annotate container element types: `Vector{Float64}(undef, n)` not `[]` when the type is known.
- Avoid untyped containers in hot loops. `Any[]` kills performance.
- Use concrete field types in structs. Avoid `struct Foo; x; end` -- use `struct Foo{T}; x::T; end`.

### Functions and Design
- Keep functions small and single-purpose. Under 30 lines where practical.
- Use clear, descriptive function names. Follow Julia naming conventions:
  - Functions: `lowercase_with_underscores()` (e.g., `process_data()`)
  - Types/Modules: `CamelCase` (e.g., `DataProcessor`)
  - Constants: `UPPERCASE` (e.g., `MAX_ITERATIONS`)
  - Boolean functions: `is_` or `has_` prefix (e.g., `is_valid()`)
- Document function signatures with docstrings:
  ```julia
  """
      compute_mean(data::Vector{Float64})::Float64

  Compute the arithmetic mean of `data`.

  # Arguments
  - `data`: Vector of numeric values (must not be empty)

  # Returns
  - Arithmetic mean as `Float64`

  # Throws
  - `ArgumentError` if `data` is empty

  # Examples
  ```julia
  julia> compute_mean([1.0, 2.0, 3.0])
  2.0
  ```
  """
  ```

### Multiple Dispatch
- Use multiple dispatch when it genuinely reduces code duplication or clarifies intent.
- Do not create excessive method specializations for types that share identical logic.
- Prefer `f(x::AbstractVector)` over separate methods for `Vector`, `SubArray`, etc., when behavior is identical.
- Use traits (small parametric types) for conditional behavior, not `isa` checks in function bodies.

### Global State
- **No global mutable state.** No `global x = 0` in module scope.
- Use `const` for module-level constants. If mutable state is needed, wrap it in a struct and pass explicitly.
- Configuration should be structs passed as arguments, not global variables.
- If constants must be computed at load time, use `const` with a type annotation.

### Arrays and Performance
- Avoid unnecessary array copies. Know when operations allocate:
  - `view(A, i:j)` instead of `A[i:j]` when you only need to read
  - `@views` macro to apply views to a block of code
  - Pre-allocate output arrays in hot loops: `result = similar(A)`
- Use broadcasting (`f.(x)`) instead of explicit loops where clear and equivalent.
- Use `map`, `filter`, `reduce` for array operations. Use generator expressions: `sum(x^2 for x in v)`.
- Use `eachindex(A)` and `axes(A)` for generic iteration, not `1:length(A)`.

### Package Usage
- Every package dependency must be justified in the Architecture Plan.
- Prefer the standard library: `LinearAlgebra`, `Statistics`, `Printf`, `Dates`, `Logging`, `UUIDs`.
- Evaluate package quality before adding: check registry age, maintenance activity, test coverage.
- Use `[compat]` bounds in `Project.toml` for all direct dependencies.
- Pin major versions. Test with `] instantiate` from a clean environment.

### Numerical Code
- Document numerical assumptions: units, scales, tolerances, convergence criteria.
- Use `eps(T)` and `floatmin(T)` appropriately when working with floating-point tolerances.
- State whether algorithms are exact or approximate.
- Handle singular/degenerate cases explicitly (empty input, zero variance, ill-conditioned matrices).
- Use `isapprox` for floating-point comparisons in tests, not exact equality.
- Document the conditioning and stability of numerical algorithms.

### Error Handling
- Use `error("descriptive message")` for unrecoverable conditions.
- Use custom exception types for library code:
  ```julia
  struct DomainError <: Exception
      msg::String
  end
  Base.showerror(io::IO, e::DomainError) = print(io, "DomainError: ", e.msg)
  ```
- Return `nothing` or a sentinel value for expected non-existence (e.g., `findfirst` returns `nothing` if not found).
- Use `ArgumentError`, `DomainError`, `BoundsError` from Base where appropriate.

---

## Julia Testing

### Test Framework
- Use Julia's built-in `Test` standard library. No external test framework needed.
- Test file structure:
  ```
  test/
    runtests.jl        -- Entry point, includes all test files
    test_core.jl       -- Core logic tests
    test_utils.jl      -- Utility function tests
    test_edge_cases.jl -- Edge case and error tests
  ```
- `runtests.jl`:
  ```julia
  using Test, MyPackage
  include("test_core.jl")
  include("test_utils.jl")
  ```

### Test Practices
- Use `@test` for boolean assertions.
- Use `@test_throws ExceptionType expr` for error testing.
- Use `@testset "descriptive name" begin ... end` to group tests.
- Use `@inferred f(args)` to assert type stability.
- Use `@test isapprox(a, b, rtol=1e-6)` for floating-point comparisons.
- Test edge cases: empty collections, single elements, zeros, infinities, NaN.
- Test error cases explicitly. Every `throw` or `error` must have a test.
- Use explicit test data in each testset. No shared mutable state between tests.

### Test Commands
```bash
julia --project=. -e "using Pkg; Pkg.test()"
# or from Julia REPL:
] test
```

---

## Julia Benchmarking

Use `BenchmarkTools.jl` for all performance measurement:

```julia
using BenchmarkTools

# Quick timing with allocation info
@btime my_function(\$data)

# Detailed benchmark
b = @benchmark my_function(\$data)
display(b)

# Memory-allocation sensitive benchmarks
@benchmark my_function(\$data) samples=1000 evals=100
```

### Benchmarking Rules
- Always interpolate variables with `$` in `@btime` and `@benchmark` to avoid timing the variable lookup.
- Report: minimum time, median time, mean time, memory allocations, and allocation size.
- Compare before and after in optimization. No optimization without measurement.
- Put benchmarks in `benchmark/bench.jl` using `BenchmarkTools`:
  ```julia
  using BenchmarkTools, MyPackage

  const suite = BenchmarkGroup()
  data = randn(10_000)
  suite["compute_mean"] = @benchmarkable compute_mean(\$data)
  suite["compute_std"] = @benchmarkable compute_std(\$data)

  tune!(suite)
  results = run(suite)
  display(results)
  ```

---

## Julia Project Structure Template

Use this structure unless the project demands otherwise:

```
ProjectName/
  Project.toml          -- Dependencies, compat bounds, UUID
  Manifest.toml         -- Locked dependency versions (commit this for applications)
  src/
    ProjectName.jl      -- Module definition, exports, includes
    types.jl            -- Core type definitions
    core.jl             -- Core algorithms and business logic
    utils.jl            -- Utility functions
    io.jl               -- File I/O, data loading/saving
  test/
    runtests.jl         -- Test entry point
    test_core.jl        -- Core logic tests
    test_edge_cases.jl  -- Edge case tests
    test_typestable.jl  -- Type stability tests (@inferred)
  benchmark/
    bench.jl            -- BenchmarkTools benchmarks
  examples/
    basic.jl            -- Basic usage example
    advanced.jl         -- Advanced usage example
  docs/                 -- Documentation (optional, use Documenter.jl)
  README.md             -- Setup, build, test, usage
  LICENSE
  .gitignore
```

### Project.toml Requirements

```toml
name = "ProjectName"
uuid = "..."  # Generated by Pkg
authors = ["Author Name <email@example.com>"]
version = "0.1.0"

[deps]
# Every dependency must be justified in the Architecture Plan

[compat]
# Pin major versions for reproducibility
julia = "1.9, 1.10"

[extras]
Test = "8dfed614-e22c-5e08-85e1-65c5234f0b40"
BenchmarkTools = "6e4b80f9-dd63-53aa-95a3-0cdb28fa8baf"

[targets]
test = ["Test", "BenchmarkTools"]
```

---

## Julia Review Checklist

Before marking a Julia project complete, verify every item:

### Code Quality
- [ ] `] test` passes all tests
- [ ] All functions have docstrings with arguments, returns, throws, and examples
- [ ] Naming follows Julia conventions
- [ ] No global mutable state
- [ ] No dead code or unused variables

### Type Stability
- [ ] `@code_warntype` shows no red (unstable) types in hot paths
- [ ] `@inferred` tests pass for performance-critical functions
- [ ] Container types are explicit (not `Any[]` in hot code)
- [ ] Struct field types are concrete or parametric

### Testing
- [ ] All exported functions have tests
- [ ] Error paths tested with `@test_throws`
- [ ] Edge cases covered (empty, zero, NaN, Inf)
- [ ] Floating-point tests use `isapprox`, not `==`
- [ ] Type stability verified with `@inferred`

### Performance
- [ ] Benchmarks exist if performance matters
- [ ] No unnecessary allocations in hot loops
- [ ] Views used instead of copies where appropriate
- [ ] Pre-allocation used in repeated operations

### Numerical Correctness
- [ ] Numerical assumptions documented (units, tolerances)
- [ ] Singular/degenerate cases handled
- [ ] Algorithm stability and conditioning documented

### Documentation
- [ ] README includes: what it does, installation, testing, usage
- [ ] Examples in `examples/` run correctly
- [ ] Docstrings are complete with `# Examples` sections

---

## Julia-Specific Decision Guide

| Decision | Prefer | Avoid |
|----------|--------|-------|
| Data structures | `StructArrays`, `NamedTuple` | Unnecessarily complex types |
| Linear algebra | `LinearAlgebra` (stdlib) | Reimplementing BLAS operations |
| I/O | `DelimitedFiles`, `JSON3`, `CSV`, `JLD2` | Manual parsing when a package exists |
| Testing | Built-in `Test` | External test frameworks |
| Benchmarking | `BenchmarkTools.jl` | `@time` in loops (compilation bias) |
| Plotting | `Makie` or `Plots` | N/A |
| Logging | `Logging` (stdlib) | `println` debugging |
| Configuration | Structs passed explicitly | Global mutable config |
| Differentiation | `ForwardDiff` or `Zygote` | Manual finite differences |
| Optimization | `Optim.jl` or `JuMP` | Manual optimizers |
