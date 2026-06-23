# Claude Code Integration Guide

How to use the Agent Swarm Programming Framework with the Claude Code CLI.

---

## Prerequisites

- Claude Code installed: `npm install -g @anthropic-ai/claude-code`
- Authenticated: `claude auth login`
- Target language toolchain installed (Rust/TypeScript/Julia/C++)

---

## Quick Start (30 Seconds)

### Step 1: Open Claude Code

```bash
cd your-project-directory
claude
```

### Step 2: Load the Framework

Copy the entire contents of `PROMPT_UNIVERSAL.md` and paste into the Claude Code chat. Press Enter.

### Step 3: Describe Your Goal

Type your request. Examples:

```
"Write a Rust CLI that converts CSV to Parquet with progress bars and error logging."
```

```
"Create a TypeScript/Express REST API for user authentication with bcrypt and JWT."
```

```
"Build a Julia notebook that loads a dataset, cleans missing values, runs PCA, and plots the first two components."
```

```
"Write a C++17 header-only ring buffer with lock-free SPSC semantics."
```

### Step 4: Execute

Claude Code triggers the full agent swarm automatically. Progress appears step-by-step.

---

## Step-by-Step Detailed Workflow

### 1. Open Claude Code

```bash
claude
```

You will see the Claude Code prompt (`>`).

### 2. Paste the Universal Prompt

Open `PROMPT_UNIVERSAL.md` in a text editor or cat it:

```bash
cat PROMPT_UNIVERSAL.md  # Copy the entire output
```

Paste into Claude Code. This loads all 11 agent definitions and the 13-step lifecycle into the context window.

**What happens:** Claude Code now understands the framework. It will respond with an acknowledgment that the swarm framework is loaded.

### 3. Describe Your Goal

After the prompt is acknowledged, describe what you want to build.

**Good goal descriptions:**
- Specify the problem, not the solution: `"I need to parse log files and extract error frequencies"` not `"Write a regex"`
- Include input/output examples if possible
- Mention constraints: performance targets, memory limits, deployment environment
- Specify if tests or benchmarks are required

**Goal description template:**

```
I need [what] that [does what].
Input: [description or example]
Output: [description or example]
Constraints: [performance, memory, dependencies, platform]
Must include: [tests, benchmarks, CLI, API, documentation]
```

### 4. Agent Execution

Claude Code executes agents in sequence. Each agent produces visible output:

```
[Goal Analyst] Analyzing goal...
[Query Logic] Evaluating language options...
[Planner] Creating development plan...
[Architect] Designing structure...
[Language Specialist] Enforcing idioms...
[Sandbox] Writing first draft...
[Test] Running tests...
[Review] Scoring: 8/10...
[Comparison] Checking goal alignment...
[Optimize] Applying improvements...
[Final Review] Gate check passed.
```

### 5. Review and Iterate

When the swarm finishes:

1. **Review the code** — Check the generated files
2. **Check the score** — Review agent scores (1-10)
3. **Run tests locally** — `cargo test`, `npm test`, `julia test/runtests.jl`, `make test`
4. **Request changes** — If something is wrong, tell Claude Code which agent to re-run

---

## How the Framework Auto-Triggers Agents

The universal prompt contains structured instructions that cause Claude Code to:

1. **Parse the goal** and classify it into a request type
2. **Run the Query Logic scoring table** against the goal's requirements
3. **Select the top-scoring language** and load its language prompt
4. **Execute each agent sequentially** with the defined input/output contract
5. **Enforce gate checks** — a step cannot proceed until the previous step's success criteria are met
6. **Loop back** when a step fails (e.g., tests fail → return to Sandbox agent)

The auto-trigger mechanism is prompt-based. Claude Code follows the structured instructions in `PROMPT_UNIVERSAL.md` which reference the agent definitions from `AGENT_SWARM.md`.

---

## Customizing Prompts Per-Project

### Method 1: Project Config File

Create `.claude-swarm-config.md` in your project root:

```markdown
# Project Overrides

## Language
Override: Rust
Reason: Team policy — all backend services use Rust

## Acceptance Criteria
- Must compile with `#![deny(warnings)]`
- All public APIs must have doc comments
- Minimum 80% test coverage

## Conventions
- Module naming: snake_case
- Error handling: use thiserror, never unwrap in production code
- Logging: use tracing, log at info level for operations

## Dependencies
- tokio = "1"
- tracing = "0.1"
- thiserror = "1"
```

Paste this file's contents immediately after the universal prompt and before your goal description.

### Method 2: Inline Overrides

Add overrides directly in your goal description:

```
Build a web scraper in Rust.
OVERRIDE: Use reqwest instead of hyper.
OVERRIDE: Target Rust 1.75 minimum.
OVERRIDE: Add retry logic with exponential backoff.
```

### Method 3: Custom Prompt File

Create a custom prompt that includes the universal prompt plus your additions:

```bash
cat PROMPT_UNIVERSAL.md my-overrides.md > my-prompt.md
```

Paste `my-prompt.md` into Claude Code.

---

## How to Override Language Decisions

### Override Before Execution

Tell Claude Code to skip language analysis:

```
Use TypeScript. Skip language analysis.
Goal: [your goal]
```

### Override During Execution

If you disagree with the Query Logic Agent's recommendation, interrupt and specify:

```
OVERRIDE LANGUAGE: Julia
Rationale: Our team has existing Julia code for this domain.
```

The swarm will restart from Step 3 with the new language.

### Force Language with No Analysis

Add to your `.claude-swarm-config.md`:

```markdown
## Language
Force: C++
Skip analysis: true
```

---

## How to Add New Languages

### Step 1: Create the Language Prompt

Create `PROMPT_<LANGUAGE>.md`:

```markdown
# [Language] Specialist Prompt

## Language Rules
[Your rules here]

## Standard Library Preferences
[Preferred libraries and patterns]

## Anti-Patterns to Reject
[Common mistakes to avoid]

## Testing Framework
[How to write tests in this language]

## Build/Run Commands
[How to compile and execute]
```

### Step 2: Add to Query Logic

Edit `QUERY_LOGIC.md`. Add a column for your language in the scoring table and define scores for each category.

### Step 3: Add to Decision Matrix

Edit `LANGUAGE_DECISION_MATRIX.md`. Add:
- Column in the quick-reference table
- Full language profile (strengths, weaknesses, ideal use cases)
- Branch in the decision flowchart

### Step 4: Define the Specialist Agent

Edit `AGENT_SWARM.md`. Add a subsection under **Language Specialist Agent** for your language with enforcement rules.

### Step 5: Test

Paste the universal prompt plus your new language prompt into Claude Code and verify the swarm executes correctly.

---

## Tips for Best Results

### Write Clear Goals

| Bad | Good |
|-----|------|
| "Write a Rust program" | "Write a Rust CLI that reads stdin, counts word frequency, and outputs top 10 words to stdout" |
| "Make a web API" | "Create a TypeScript Express API with GET/POST /users endpoints, SQLite storage, and input validation" |
| "Do data analysis" | "Julia script: load CSV, compute correlation matrix, handle missing values, save results to JSON" |

### Include Constraints Up Front

Mention these if they matter:
- **Performance targets** ("must handle 10k requests/second")
- **Memory limits** ("must run on a 512MB container")
- **Dependencies** ("no external crates except tokio")
- **Platform** ("must compile on Windows")
- **Rust version** / **Node version** / **Julia version** / **C++ standard**

### Review Agent Scores

Every code review produces a score 1-10. If any agent scores below 7, ask Claude Code to re-run that agent:

```
The Best Practices Review scored 5/10. Re-run the review agent and fix all issues.
```

### Iterate on Test Failures

If the Test Agent finds failures, the swarm should automatically loop back. If it doesn't:

```
Test failures found. Return to Sandbox Coding Agent and fix.
```

### Save Successful Prompts

When you get a good result, save the conversation. Reuse successful prompt patterns for similar tasks.

---

## Troubleshooting

### Problem: Claude Code doesn't follow the framework

**Cause:** The prompt was too long or got truncated.

**Fix:**
1. Start a fresh Claude Code session (`/clear` or restart)
2. Paste `PROMPT_UNIVERSAL.md` in chunks if needed
3. Confirm Claude acknowledges the framework before describing your goal

### Problem: Wrong language selected

**Cause:** Query Logic Agent misinterpreted requirements.

**Fix:**
1. Add more specific constraints to your goal
2. Use language override: `"Use Rust regardless of analysis"`
3. Check `QUERY_LOGIC.md` to understand the scoring

### Problem: Tests fail repeatedly

**Cause:** The goal may be underspecified or contradictory.

**Fix:**
1. Restate the goal with clearer input/output examples
2. Add explicit edge cases you want handled
3. Start with a simpler version of the goal, then add features

### Problem: Code is over-engineered

**Cause:** The Architect or Sandbox agent added unnecessary abstractions.

**Fix:**
1. Add to your goal: `"Keep it simple. No unnecessary abstractions."`
2. After the Planner step, review the plan and say: `"Remove steps 3 and 4. Simpler approach."`
3. Add to `.claude-swarm-config.md`: `Complexity: minimal`

### Problem: Agent scores are consistently low

**Cause:** The language prompt may be incomplete or the goal may be too vague.

**Fix:**
1. Check that the language-specific prompt loaded correctly
2. Make your goal more specific
3. Add acceptance criteria to your `.claude-swarm-config.md`

### Problem: Claude Code runs out of context

**Cause:** Very large projects exceed the context window.

**Fix:**
1. Break the goal into smaller sub-goals
2. Run the swarm for one module at a time
3. Use `/clear` between major phases

---

## Summary

| Task | Command/Action |
|------|---------------|
| Open Claude Code | `claude` |
| Load framework | Paste `PROMPT_UNIVERSAL.md` |
| Describe goal | Type your request |
| Override language | Add `"Use [language]"` to goal |
| Add project config | Paste `.claude-swarm-config.md` after universal prompt |
| Fix test failures | Ask Claude to re-run Sandbox agent |
| Request re-review | Ask Claude to re-run specific agent |

**Next:** Read [AGENT_SWARM.md](AGENT_SWARM.md) to understand what each agent does in detail.