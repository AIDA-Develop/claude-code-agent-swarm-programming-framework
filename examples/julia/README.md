# Julia Example — Placeholder

This directory is a placeholder for a future Julia example project generated using the Agent Swarm Programming Framework.

## Intended Project Type

A Julia script or package demonstrating the framework's output for a numerical computing or data analysis task — for example, an ODE solver, a statistical analysis pipeline, a PCA workflow, or a linear regression tool with plots.

## How to Generate Your Own

1. Open Claude Code in any directory: `claude`
2. Paste the contents of `PROMPT_UNIVERSAL.md` from the repo root
3. Describe a Julia goal, for example:

   > "Build a Julia script that loads a CSV dataset, computes a correlation matrix, handles missing values, runs PCA, and saves a plot of the first two principal components."

4. The swarm selects Julia, plans, architects, codes, tests, reviews, optimizes, and delivers production-ready code.

## What to Expect

The generated project will include:
- `src/` with type-stable, idiomatic Julia source code
- `test/runtests.jl` with a Test.jl test suite
- `Project.toml` and `Manifest.toml` with pinned dependencies
- Usage instructions and known limitations
- A quality score (1–10) from the Best Practices Review Agent

## Reference Files

- [`PROMPT_JULIA.md`](../../PROMPT_JULIA.md) — Julia-specific rules injected during generation
- [`BEST_PRACTICES_JULIA.md`](../../BEST_PRACTICES_JULIA.md) — Julia best practices the Review Agent enforces
- [`EXAMPLE_USER_QUERIES.md`](../../EXAMPLE_USER_QUERIES.md) — Worked Julia query examples
