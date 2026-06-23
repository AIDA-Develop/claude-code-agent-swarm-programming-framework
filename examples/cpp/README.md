# C++ Example — Placeholder

This directory is a placeholder for a future C++ example project generated using the Agent Swarm Programming Framework.

## Intended Project Type

A C++ library or systems component demonstrating the framework's output for a performance-critical or hardware-adjacent task — for example, a lock-free concurrent queue, a ring buffer, an embedded firmware module, or a high-throughput data structure.

## How to Generate Your Own

1. Open Claude Code in any directory: `claude`
2. Paste the contents of `PROMPT_UNIVERSAL.md` from the repo root
3. Describe a C++ goal, for example:

   > "Write a C++20 header-only library implementing a lock-free SPSC ring buffer with configurable capacity, move semantics, and a Catch2 test suite."

4. The swarm selects C++, plans, architects, codes, tests, reviews, optimizes, and delivers production-ready code.

## What to Expect

The generated project will include:
- `include/` or `src/` with modern C++ (C++17/20) source code
- `tests/` with a Catch2 or GoogleTest suite
- `CMakeLists.txt` with sanitizer and warning configurations
- Usage instructions and known limitations
- A quality score (1–10) from the Best Practices Review Agent

## Reference Files

- [`PROMPT_CPP.md`](../../PROMPT_CPP.md) — C++-specific rules injected during generation
- [`BEST_PRACTICES_CPP.md`](../../BEST_PRACTICES_CPP.md) — C++ best practices the Review Agent enforces
- [`EXAMPLE_USER_QUERIES.md`](../../EXAMPLE_USER_QUERIES.md) — Worked C++ query examples
