# Claude Code Agent Swarm Programming Framework

> A strict, repeatable, multi-agent workflow for AI-assisted programming. Claude Code writes, tests, reviews, optimizes, and finalizes production-ready code across Rust, TypeScript, Julia, C++, and Python.

---

## Overview

This framework is a **repeatable, copy-paste workflow** designed for Claude Code. It treats AI-assisted programming as a structured engineering process: a swarm of specialized agent roles collaborate to turn your goal into tested, reviewed, optimized code.

No guesswork. No "write once and hope." Every piece of code passes through **13 lifecycle steps**, **11 specialized agent roles**, and **5 supported languages** before it reaches you.

### What This Framework Does

1. You describe your goal in plain English
2. The **Query Logic Agent** selects the optimal language from Rust, TypeScript, Julia, C++, or Python
3. A **Planner Agent** breaks the work into phases, files, and tests
4. An **Architect Agent** designs the structure and patterns
5. A **Language Specialist Agent** enforces idiomatic, best-practice code
6. A **Sandbox Coding Agent** writes the first working version
7. A **Test Agent** verifies correctness with happy paths, edges, and regressions
8. A **Best Practices Review Agent** scores the code 1-10 and mandates fixes
9. A **Goal Comparison Agent** ensures the output matches your original request
10. An **Optimization Agent** improves correctness, safety, performance, and readability
11. A **Final Review Agent** acts as the last gate before delivery

### The 5 Supported Languages

| Language | Best For |
|----------|----------|
| **Rust** | Systems programming, performance-critical code, memory safety, concurrency |
| **TypeScript** | Web apps, backend APIs, AI integrations, full-stack development |
| **Julia** | Numerical computing, data analysis, scientific computing, AI/ML inference |
| **C++** | Hardware-level control, embedded systems, game engines, extreme performance |
| **Python** | AI/ML integration, automation, data pipelines, rapid prototyping, backend glue |

---

## Quick Start

### 1. Copy the Universal Prompt

Open `PROMPT_UNIVERSAL.md` in this repo. Copy the entire contents.

### 2. Open Claude Code

```bash
claude
```

### 3. Paste the Prompt

Paste the universal prompt into the chat. Claude Code now has the full framework loaded.

### 4. Describe Your Goal

Tell Claude what you want to build. Be specific. Examples:

> "Write a Rust CLI tool that parses JSON logs and outputs aggregate statistics to CSV."

> "Create a TypeScript Express API with JWT auth and a PostgreSQL connection pool."

> "Build a Julia script that runs linear regression on a CSV dataset and plots results."

> "Write a C++ header-only library for lock-free concurrent queues."

> "Build a Python service that ingests CSV data, validates it with pydantic, and exposes a FastAPI endpoint with tests."

### 5. Let the Swarm Work

Claude Code will automatically trigger each agent role in sequence. You'll see output from each step:

```
[Goal Analyst] Restating goal and identifying requirements...
[Query Logic] Analyzing requirements for optimal language selection...
[Planner] Creating phased development plan...
[Architect] Designing project structure...
[Language Specialist] Enforcing Rust best practices...
[Sandbox] Writing first working implementation...
[Test] Running test suite...
[Review] Scoring code quality: 8/10...
[Comparison] Checking against original goal...
[Optimize] Applying optimizations...
[Final Review] Gate check passed.
```

### 6. Receive Final Code

The final output includes:
- Complete source code
- Test suite with coverage
- Usage instructions
- Known limitations
- Quality score (1-10)

---

## Repository File Tree

```
claude-code-agent-swarm-programming-framework/
├── LICENSE
├── README.md                           # This file — overview and quick start
├── CLAUDE.md                           # Claude Code integration guide
├── AGENT_SWARM.md                      # Complete agent role definitions (11 agents)
├── WORKFLOW.md                         # 13-step lifecycle workflow
├── QUERY_LOGIC.md                      # Query analysis and language selection framework
├── LANGUAGE_DECISION_MATRIX.md         # Quick-reference language decision matrix
├── BEST_PRACTICES_RUST.md              # Rust best practices reference
├── BEST_PRACTICES_TYPESCRIPT.md        # TypeScript best practices reference
├── BEST_PRACTICES_JULIA.md             # Julia best practices reference
├── BEST_PRACTICES_CPP.md               # C++ best practices reference
├── BEST_PRACTICES_PYTHON.md            # Python best practices reference
├── PROMPT_UNIVERSAL.md                 # Copy-paste universal prompt for Claude Code
├── PROMPT_RUST.md                      # Rust-specific prompt override
├── PROMPT_TYPESCRIPT.md                # TypeScript-specific prompt override
├── PROMPT_JULIA.md                     # Julia-specific prompt override
├── PROMPT_CPP.md                       # C++-specific prompt override
├── PROMPT_PYTHON.md                    # Python-specific prompt override
├── REVIEW_CHECKLIST.md                 # Best practices review checklist
├── TESTING_CHECKLIST.md                # Testing checklist
├── SECURITY_CHECKLIST.md               # Security checklist
├── OPTIMIZATION_CHECKLIST.md           # Optimization checklist
├── FINAL_OUTPUT_TEMPLATE.md            # Final output delivery template
├── EXAMPLE_USER_QUERIES.md             # 8 worked example user queries
└── examples/                           # Placeholder example projects
    ├── rust/
    │   └── README.md
    ├── typescript/
    │   └── README.md
    ├── julia/
    │   └── README.md
    └── cpp/
        └── README.md
```

---

## File Descriptions

### Core Documentation

| File | Purpose |
|------|---------|
| `README.md` | Overview, quick start, file tree, and framework introduction |
| `CLAUDE.md` | Step-by-step guide for using this framework with Claude Code CLI |
| `AGENT_SWARM.md` | All 11 agent roles defined with responsibilities, inputs, outputs, and success criteria |
| `WORKFLOW.md` | The 13-step lifecycle from goal to final delivery, with ASCII diagrams and error handling |
| `QUERY_LOGIC.md` | Scoring methodology, weighted decision tables, and worked examples for language selection |
| `LANGUAGE_DECISION_MATRIX.md` | Quick-lookup matrix, per-language profiles, decision flowchart, and override rules |

### Best Practices References

| File | Purpose |
|------|---------|
| `BEST_PRACTICES_RUST.md` | Rust idioms, project setup, required tooling, common mistakes, and a minimal example |
| `BEST_PRACTICES_TYPESCRIPT.md` | TypeScript config, code style, testing, ESLint/Prettier setup, and a minimal example |
| `BEST_PRACTICES_JULIA.md` | Julia type stability, multiple dispatch, testing, benchmarking, and a minimal example |
| `BEST_PRACTICES_CPP.md` | C++ RAII, smart pointers, CMake setup, sanitizer builds, and a minimal example |
| `BEST_PRACTICES_PYTHON.md` | Python type hints, src layout, ruff/mypy/pytest tooling, common mistakes, and a minimal example |

### Prompt Files

| File | Purpose |
|------|---------|
| `PROMPT_UNIVERSAL.md` | The main copy-paste prompt — paste this into Claude Code to activate the framework |
| `PROMPT_RUST.md` | Override/additions when the selected language is Rust |
| `PROMPT_TYPESCRIPT.md` | Override/additions when the selected language is TypeScript |
| `PROMPT_JULIA.md` | Override/additions when the selected language is Julia |
| `PROMPT_CPP.md` | Override/additions when the selected language is C++ |
| `PROMPT_PYTHON.md` | Override/additions when the selected language is Python |

### Checklists and Templates

| File | Purpose |
|------|---------|
| `REVIEW_CHECKLIST.md` | Weighted review checklist used by the Best Practices Review Agent |
| `TESTING_CHECKLIST.md` | Test category requirements, coverage targets, and per-language testing guides |
| `SECURITY_CHECKLIST.md` | Security risk classification and per-language vulnerability checklist |
| `OPTIMIZATION_CHECKLIST.md` | Five-tier optimization priority order and anti-patterns to avoid |
| `FINAL_OUTPUT_TEMPLATE.md` | 14-section delivery template used by the Final Review Agent |
| `EXAMPLE_USER_QUERIES.md` | 8 worked example goals covering the supported languages and request types |

### Examples Directory

| Directory | Purpose |
|-----------|---------|
| `examples/rust/` | Placeholder for a future Rust example project |
| `examples/typescript/` | Placeholder for a future TypeScript example project |
| `examples/julia/` | Placeholder for a future Julia example project |
| `examples/cpp/` | Placeholder for a future C++ example project |

---

## The 13-Step Lifecycle

Every project flows through these 13 steps in order. Steps may loop back when issues are found.

```
 1. Understand the user's goal
 2. Analyze the query requirements
 3. Decide which language is best suited
 4. Create a plan
 5. Architect the solution
 6. Sandbox the code
 7. Write the first implementation
 8. Test the implementation
 9. Review against language-specific best practices
10. Compare the code against the original goal
11. Optimize for correctness, safety, performance, readability, maintainability
12. Run a final review
13. Present final code with explanation, usage instructions, known limitations
```

See [WORKFLOW.md](WORKFLOW.md) for the full specification of each step.

---

## The 11 Agent Roles

| # | Agent | What It Does |
|---|-------|-------------|
| 1 | **Goal Analyst** | Restates goals, finds hidden requirements, classifies request type, produces acceptance checklist |
| 2 | **Query Logic** | Analyzes runtime, performance, safety, concurrency needs; selects the best language from the 5 options |
| 3 | **Planner** | Converts the goal into a step-by-step plan with phases, milestones, files, tests, and completion criteria |
| 4 | **Architect** | Designs project structure, patterns, libraries, separation of concerns, error handling, testing strategy |
| 5 | **Language Specialist** | Enforces idiomatic code for the selected language (with 5 sub-specialists for Rust, TS, Julia, C++, Python) |
| 6 | **Sandbox Coding** | Writes the first working version — minimal, complete, runnable, with tests from the start |
| 7 | **Test Agent** | Writes and runs happy path, edge case, invalid input, regression, and performance tests |
| 8 | **Best Practices Review** | Scores code 1-10 on language best practices, security, maintainability, performance, naming, structure |
| 9 | **Goal Comparison** | Verifies the output matches the original goal and acceptance criteria |
| 10 | **Optimization** | Improves code only after tests pass — for correctness, simplicity, safety, performance, usability |
| 11 | **Final Review** | Last gate: matches goal, passes tests, follows best practices, clear usage, no security issues |

See [AGENT_SWARM.md](AGENT_SWARM.md) for complete agent specifications.

---

## Example Usage

### Starting a New Project

1. **Open your terminal** and navigate to your project directory:
   ```bash
   mkdir my-project && cd my-project
   claude
   ```

2. **Paste the universal prompt.** Open `PROMPT_UNIVERSAL.md` from this repo and copy-paste the entire file into Claude Code.

3. **Describe your goal.** After the prompt is loaded, type your request. For example:
   > "I need a Rust library that implements a thread-safe LRU cache with TTL expiration. It should have async support and benchmarks."

4. **Watch the swarm execute.** Claude Code will output progress from each agent:
   ```
   === [1/11] Goal Analyst ===
   Goal: Thread-safe LRU cache with TTL, async support, benchmarks
   Classification: Library
   Hidden requirements: eviction callbacks, memory pressure handling
   Acceptance checklist: [7 items]

   === [2/11] Query Logic ===
   Scoring analysis complete. Recommended: Rust (score 94)
   Rationale: Memory safety + concurrency + systems programming needs
   ...
   ```

5. **Review the output.** When complete, you'll have:
   - A new `src/` directory with source files
   - A `tests/` directory with test files
   - `Cargo.toml` (or equivalent)
   - Usage instructions in the chat output
   - Quality score and known limitations

### Customizing Per-Project

To override language selection, add this to your goal description:

> "...Force language: TypeScript. Rationale: team expertise and deployment target."

To skip language analysis and use your own preference:

> "...Use Rust regardless of analysis. Our team requires it."

See [CLAUDE.md](CLAUDE.md) for full customization options.

---

## How to Contribute / Customize

### Adding a New Language

1. Create `PROMPT_<LANGUAGE>.md` following the pattern of existing language prompts
2. Add the language column to the scoring table in `QUERY_LOGIC.md`
3. Add the language profile to `LANGUAGE_DECISION_MATRIX.md`
4. Add a Language Specialist sub-agent section in `AGENT_SWARM.md`
5. Update the README language table

### Customizing Agent Behavior

Edit the agent definitions in `AGENT_SWARM.md`. Each agent has clear input/output boundaries — modify the output criteria or add new checks without breaking the workflow.

### Adding New Agent Roles

Insert the new agent into the workflow in `WORKFLOW.md`, then define it in `AGENT_SWARM.md`. Update step numbers and agent references throughout.

### Project-Specific Overrides

Create a `.claude-swarm-config.md` in your project root. This file can:
- Override the default language
- Add custom acceptance criteria
- Specify team conventions (naming, structure)
- Pin dependency versions

---

## Requirements

- [Claude Code](https://claude.ai/code) installed and authenticated
- The target language toolchain installed (e.g., `rustc`, `node`, `julia`, `g++`)

---

## License

MIT License — see LICENSE file for details.

---

## Summary

This framework turns AI-assisted programming from ad-hoc prompting into a **structured, repeatable engineering process**. Copy `PROMPT_UNIVERSAL.md`, paste it into Claude Code, describe your goal, and let 11 specialized agents deliver production-ready code.

**Start now:** Open `PROMPT_UNIVERSAL.md` and copy it into Claude Code.