# Julia Best Practices for Claude Code Agent Swarm

> One-sentence rule: **Write type-stable, readable code; let multiple dispatch do the heavy lifting.**

---

## Philosophy

- **Multiple dispatch** — design functions around operations, not object hierarchies. The method table is your friend.
- **Type stability** — write code where the compiler can infer output types from input types. `@code_warntype` is your first profiling tool.
- **Math-oriented clarity** — Julia shines at numerical computing. Prefer readable, math-expressive code over micro-optimized loops unless performance demands it.

---

## Project Setup

### Create and Activate a New Environment

```julia
using Pkg
Pkg.activate(".")       # Creates/uses Project.toml in current directory
Pkg.add("SomePackage")
Pkg.instantiate()        # Install all declared dependencies
```

### Required Commands

| Command | Purpose |
|---------|---------|
| `using Pkg; Pkg.activate("."); Pkg.instantiate()` | Activate environment and install dependencies |
| `Pkg.test()` | Run all tests in `test/runtests.jl` |

### Optional: Benchmarking

```julia
using BenchmarkTools
@benchmark my_function($(rand(1000)))   # $ interpolates to avoid timing overhead
```

---

## Code Style Rules

### Idiomatic Julia

- **Use 4-space indentation.**
- **No semicolons at end of lines** unless suppressing output in the REPL.
- **Module names**: `PascalCase` (e.g., `MyPackage`).
- **Function names**: `snake_case` (e.g., `compute_gradient`).
- **Type names**: `PascalCase` (e.g., `LinearModel`).
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `DEFAULT_TOLERANCE`).
- **File names**: `snake_case.jl` (e.g., `data_loader.jl`).
- **Functions that mutate their arguments end with `!`** (e.g., `sort!`, `normalize!`).

### Type-Stable Code

- **Avoid containers with abstract element types.** This is the #1 performance killer:

```julia
# Bad — Vector{Any}, slow
x = []
for i in 1:1000
    push!(x, i * 2.0)
end

# Good — Vector{Float64}, fast
x = Float64[]
for i in 1:1000
    push!(x, i * 2.0)
end

# Good — pre-allocated, type-known
x = zeros(1000)
for i in eachindex(x)
    x[i] = i * 2.0
end
```

- **Check type stability** before you profile:

```julia
@code_warntype my_function(args...)   # red = type instability
```

### Clear Function Boundaries

- **One function, one responsibility.**
- **Document input assumptions** — expected ranges, units, array shapes.
- **Return consistent types** from the same function. If you need variant returns, use multiple dispatch or a wrapper type.

### Multiple Dispatch

- **Use multiple dispatch when it simplifies design** — define methods on shared functions for different type combinations.
- **Do not force every problem into dispatch.** Plain functions are fine.

```julia
abstract type Shape end
struct Circle <: Shape; radius::Float64; end
struct Rectangle <: Shape; width::Float64; height::Float64; end

# Multiple dispatch — same function, different types
area(c::Circle) = π * c.radius^2
area(r::Rectangle) = r.width * r.height
area(shapes::Vector{<:Shape}) = sum(area, shapes)
```

### Avoid Global Mutable State

- **Never use untyped global variables in performance-critical code.**
- Pass data as arguments. Use `const` for true constants:

```julia
const DEFAULT_EPSILON = 1e-8   # OK — typed constant

# Bad for performance
x = 0.0
for i in 1:1_000_000
    global x += sin(i)         # untyped global — slow
end

# Good
function compute_sum(n::Int)::Float64
    x = 0.0
    for i in 1:n
        x += sin(i)
    end
    return x
end
```

### Packages and Dependencies

- **Use packages only when justified.** Julia's standard library is substantial (`LinearAlgebra`, `Statistics`, `Printf`, `Dates`, `JSON3`, etc.).
- **Add to `Project.toml`, not ad-hoc installs.** Every dependency is a reproducibility contract.

### Benchmarking

- **Benchmark before optimizing.** Use `BenchmarkTools.jl` for reliable measurements:

```julia
using BenchmarkTools

# Use $ to interpolate values (prevents timing the construction)
@benchmark sum($(rand(10000)))

# Compare implementations
@btime slow_version($data)
@btime fast_version($data)
```

### Documentation

- **Document numerical assumptions** — units, expected ranges, precision requirements.
- Use docstrings for all public functions:

```julia
"""
    solve_quadratic(a, b, c)

Solve `ax^2 + bx + c = 0` for real roots.

# Arguments
- `a::Real`: quadratic coefficient (must be non-zero)
- `b::Real`: linear coefficient
- `c::Real`: constant term

# Returns
- `Tuple{Float64, Float64}`: the two real roots, or throws `DomainError` if discriminant < 0

# Examples
```jldoctest
julia> solve_quadratic(1, -3, 2)
(1.0, 2.0)
```
"""
function solve_quadratic(a::Real, b::Real, c::Real)
    a == 0 && throw(ArgumentError("coefficient a must be non-zero"))
    discriminant = b^2 - 4a * c
    discriminant < 0 && throw(DomainError(discriminant, "no real roots"))
    sqrt_d = sqrt(discriminant)
    return ((-b - sqrt_d) / 2a, (-b + sqrt_d) / 2a)
end
```

### Array Operations

- **Avoid copying large arrays unnecessarily.** Use views, broadcasting, and in-place operations.

```julia
# Bad — allocates new array
y = x[1:1000] .+ 1

# Good — view, no copy
y = @view x[1:1000] .+ 1

# Good — in-place, no allocation
y .= x[1:1000] .+ 1

# Good — explicit in-place mutation
function add_one!(x::AbstractVector)
    @inbounds for i in eachindex(x)
        x[i] += 1
    end
end
```

### Performance Tradeoffs

- **Make tradeoffs explicit.** If you choose a less readable implementation for speed, add a comment:

```julia
# Using loop fusion instead of chained broadcasts to avoid intermediate allocations
@inbounds for i in eachindex(a, b, c)
    c[i] = a[i] * b[i] + sqrt(a[i])
end
```

---

## Testing Rules

- **Tests live in `test/runtests.jl`.**
- Use the built-in `Test` module:

```julia
using Test
using MyPackage

@testset "math utilities" begin
    @test solve_quadratic(1, -3, 2) == (1.0, 2.0)
    @test_throws ArgumentError solve_quadratic(0, 1, 1)
    @test_throws DomainError solve_quadratic(1, 0, 1)
    @test solve_quadratic(1, 0, -1) ≈ (-1.0, 1.0)   # approximate equality
end
```

- Run tests with `Pkg.test()` or `] test` in the REPL.
- Group related tests in `@testset` blocks for clear output.
- Use `@test_throws` for expected exceptions.
- Use `≈` (`isapprox`) for floating-point comparisons.

---

## Project Structure Template

```
MyPackage/
├── Project.toml
├── README.md
├── src/
│   ├── MyPackage.jl       # module entry point
│   ├── core.jl            # core algorithms
│   ├── utils.jl           # helper functions
│   └── io.jl              # data loading/saving
├── test/
│   ├── runtests.jl        # test entry point
│   ├── test_core.jl
│   └── test_utils.jl
├── docs/
│   └── make.jl
├── benchmarks/
│   └── bench_core.jl
└── .github/
    └── workflows/
        └── ci.yml
```

**`src/MyPackage.jl`**:

```julia
module MyPackage

using LinearAlgebra

export solve_quadratic, normalize!

include("core.jl")
include("utils.jl")

end
```

**`Project.toml`** (relevant fields):

```toml
name = "MyPackage"
uuid = "..."               # generated by Pkg
authors = ["..."]
version = "0.1.0"

[deps]
LinearAlgebra = "37e2e46d-f89d-539d-b4ee-838fcccc9c8e"

[compat]
julia = "1.9"

[extras]
Test = "8dfed614-e22c-5e08-85e1-65c5234f0b40"

[targets]
test = ["Test"]
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Untyped global variables | Type instability, major slowdown | Pass as arguments or use `const` |
| `Vector{Any}` or `Array{Any}` | Every element is boxed, slow | Declare concrete element types |
| Writing C-style loops everywhere | Unnecessary in Julia | Use vectorized operations and broadcasting |
| Ignoring `@code_warntype` | Type instabilities go unnoticed | Check before profiling |
| Excessive type annotations on locals | Doesn't help, reduces clarity | Annotate function signatures only |
| Using `@inbounds` everywhere | Silent memory corruption on bugs | Use only in hot loops after correctness is proven |
| Not using `.=` for in-place assignment | Unnecessary allocations | Use in-place broadcasting |
| Loading all packages at module top level | Slow compile times, namespace pollution | Load only what you use |
| Writing one giant function | Hard to test and read | Decompose into focused functions |
| Mutating arguments without `!` | Violates Julia naming convention | Append `!` to mutating functions |

---

## Example: Minimal Correct Julia Project

```
Greet/
├── Project.toml
└── src/
    ├── Greet.jl
    └── greet.jl
└── test/
    └── runtests.jl
```

**`Project.toml`**

```toml
name = "Greet"
uuid = "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
version = "0.1.0"

[compat]
julia = "1.9"

[extras]
Test = "8dfed614-e22c-5e08-85e1-65c5234f0b40"

[targets]
test = ["Test"]
```

**`src/Greet.jl`**

```julia
module Greet

export greet

include("greet.jl")

end
```

**`src/greet.jl`**

```julia
"""
    greet(name::AbstractString)

Return a greeting string for the given `name`.

Throws `ArgumentError` if `name` is empty.

# Examples
```jldoctest
julia> greet("Ada")
"Hello, Ada!"
```
"""
function greet(name::AbstractString)::String
    isempty(name) && throw(ArgumentError("name must not be empty"))
    return "Hello, $(name)!"
end
```

**`test/runtests.jl`**

```julia
using Test
using Greet

@testset "greet" begin
    @test greet("Ada") == "Hello, Ada!"
    @test greet("World") == "Hello, World!"
    @test_throws ArgumentError greet("")
end
```

**Run the example:**

```bash
julia -e 'using Pkg; Pkg.activate("."); Pkg.test()'
```

---

## Quick Reference

```bash
# Activate environment and install deps
julia -e 'using Pkg; Pkg.activate("."); Pkg.instantiate()'

# Run tests
julia -e 'using Pkg; Pkg.test()'

# Check type stability
julia -e 'using MyPackage; @code_warntype my_function(args...)'

# Benchmark
julia -e 'using BenchmarkTools, MyPackage; @benchmark my_function($(setup_data))'

# REPL package mode shortcuts
] activate .    # activate environment
] add SomePkg   # add dependency
] test          # run tests
] status        # list dependencies
```
