# Rust Example — Placeholder

This directory is a placeholder for a future Rust example project generated using the Agent Swarm Programming Framework.

## Intended Project Type

A Rust CLI tool or library demonstrating the framework's output for a systems programming task — for example, a log parser, a concurrent queue, or a performance-critical data processing utility.

## How to Generate Your Own

1. Open Claude Code in any directory: `claude`
2. Paste the contents of `PROMPT_UNIVERSAL.md` from the repo root
3. Describe a Rust goal, for example:

   > "Write a Rust CLI that reads a log file, counts error frequencies by type, and outputs a sorted summary to stdout."

4. The swarm selects Rust, plans, architects, codes, tests, reviews, optimizes, and delivers production-ready code.

## What to Expect

The generated project will include:
- `src/` with complete, idiomatic Rust source code
- `tests/` with a full test suite (happy path, edge cases, regressions)
- `Cargo.toml` with pinned dependencies
- Usage instructions and known limitations
- A quality score (1–10) from the Best Practices Review Agent

## Reference Files

- [`PROMPT_RUST.md`](../../PROMPT_RUST.md) — Rust-specific rules injected during generation
- [`BEST_PRACTICES_RUST.md`](../../BEST_PRACTICES_RUST.md) — Rust best practices the Review Agent enforces
- [`EXAMPLE_USER_QUERIES.md`](../../EXAMPLE_USER_QUERIES.md) — Worked Rust query examples
