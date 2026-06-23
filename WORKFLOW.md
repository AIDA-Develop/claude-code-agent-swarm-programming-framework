# 13-Step Lifecycle Workflow

The complete step-by-step lifecycle that every project follows from goal description to final delivery.

---

## Workflow Diagram (ASCII)

```
  USER INPUT
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 1: Understand the user's goal                                         │
│  Agent: Goal Analyst                                                        │
│  Output: Restated goal, hidden requirements, acceptance checklist           │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 2: Analyze the query requirements                                     │
│  Agent: Goal Analyst (continued)                                            │
│  Output: Classified request type, constraints, quality criteria             │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 3: Decide which language is best suited                               │
│  Agent: Query Logic Agent                                                   │
│  Output: Language recommendation with scoring matrix and rationale          │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │ (override possible)
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 4: Create a plan                                                      │
│  Agent: Planner Agent                                                       │
│  Output: Phased development plan with milestones, files, tests              │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 5: Architect the solution                                             │
│  Agent: Architect Agent                                                     │
│  Output: Project structure, patterns, libraries, module design              │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 6: Sandbox the code                                                   │
│  Agent: Language Specialist + Sandbox Coding Agent                          │
│  Output: Language audit + first working implementation                      │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 7: Write the first implementation                                     │
│  Agent: Sandbox Coding Agent                                                │
│  Output: Complete source code, tests, build config, examples                │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  STEP 8: Test the implementation                                            │
│  Agent: Test Agent                                                          │
│  Output: Test suite, coverage report, pass/fail results                     │
└───────────────────────────────┬─────────────────────────────────────────────┘
                                │ (all tests pass?)
                    ┌───────────┴───────────┐
                    │ FAIL                  │ PASS
                    ▼                       ▼
        ┌──────────────────────┐  ┌──────────────────────────────────────────┐
        │ Return to Step 6     │  │ STEP 9: Review against language-specific │
        │ Fix and re-test      │  │ best practices                           │
        └──────────────────────┘  │ Agent: Best Practices Review Agent       │
                                  │ Output: Score card, required fixes       │
                                  └───────────────┬──────────────────────────┘
                                                  │ (score >= 7/10?)
                                    ┌─────────────┴─────────────┐
                                    │ NO                       │ YES
                                    ▼                          ▼
                        ┌──────────────────────┐  ┌──────────────────────────────┐
                        │ Return to Step 6     │  │ STEP 10: Compare against     │
                        │ Fix all required     │  │ original goal                │
                        │ issues               │  │ Agent: Goal Comparison Agent │
                        └──────────────────────┘  │ Output: Match/mismatch report│
                                                  └──────────────┬───────────────┘
                                                                 │ (matches goal?)
                                                   ┌─────────────┴─────────────┐
                                                   │ NO                       │ YES
                                                   ▼                          ▼
                                       ┌──────────────────────┐  ┌──────────────────────────┐
                                       │ Return to Step 4     │  │ STEP 11: Optimize for    │
                                       │ Re-plan and re-code  │  │ correctness, safety,     │
                                       │                      │  │ performance, readability │
                                       └──────────────────────┘  │ Agent: Optimization Agent│
                                                                  │ Output: Optimization log │
                                                                  └──────────────┬───────────┘
                                                                                 │
                                                                                 ▼
                                                                  ┌──────────────────────────┐
                                                                  │ STEP 12: Run a final     │
                                                                  │ review                   │
                                                                  │ Agent: Final Review Agent│
                                                                  │ Output: Gate check       │
                                                                  └──────────────┬───────────┘
                                                                                 │ (all gates pass?)
                                                                     ┌───────────┴───────────┐
                                                                     │ FAIL                  │ PASS
                                                                     ▼                       ▼
                                                          ┌──────────────────────┐  ┌──────────────────────┐
                                                          │ Return to failed     │  │ STEP 13: Present     │
                                                          │ agent's step         │  │ final code           │
                                                          └──────────────────────┘  └──────────────────────┘
                                                                                               │
                                                                                               ▼
                                                                                    ┌──────────────────────┐
                                                                                    │ FINAL DELIVERY       │
                                                                                    │ - Source code        │
                                                                                    │ - Test suite         │
                                                                                    │ - Usage instructions │
                                                                                    │ - Known limitations  │
                                                                                    │ - Quality score      │
                                                                                    └──────────────────────┘
```

---

## Step Details

---

### Step 1: Understand the User's Goal

**Description:**  
Parse the user's natural language goal. Extract explicit requirements, stated constraints, and implied context. Restate the goal in structured, unambiguous language that both human and AI can validate against.

**Agent Responsible:** Goal Analyst Agent

**Input:**
- User's raw goal description (text)
- `.claude-swarm-config.md` (if present)
- Conversation context (if continuing a session)

**Output:**
- Restated goal (structured text)
- Hidden requirements list
- Missing requirements list
- Acceptance checklist (5-15 items)

**What "Done" Looks Like:**  
The restated goal is specific enough that a developer could read it and know exactly what to build. At least one hidden requirement has been identified. The acceptance checklist has verifiable criteria.

---

### Step 2: Analyze the Query Requirements

**Description:**  
Classify the request type and extract all technical requirements: performance needs, safety requirements, concurrency model, deployment target, team constraints, error tolerance, and security posture.

**Agent Responsible:** Goal Analyst Agent (continued)

**Input:**
- Restated goal from Step 1
- User's original description

**Output:**
- Request type classification (one of 13 types)
- Technical requirements summary
- Constraint list
- Quality criteria

**What "Done" Looks Like:**  
The request type is unambiguously classified. Every constraint is documented. The quality criteria define how good the solution needs to be.

---

### Step 3: Decide Which Language is Best Suited

**Description:**  
Score each of the 5 supported languages across 14+ dimensions relevant to this specific goal. Apply weights based on the goal's requirements. Recommend exactly one language. If a language override exists in config, apply it and document why.

**Agent Responsible:** Query Logic Agent

**Input:**
- Restated goal
- Request type classification
- Hidden/missing requirements
- Technical requirements
- `.claude-swarm-config.md` language overrides

**Output:**
- 14x5 scoring matrix
- Weight assignments with justification
- Weighted totals per language
- Single language recommendation (High/Medium/Low confidence)
- Rejection rationale for each non-recommended language

**What "Done" Looks Like:**  
One language is clearly recommended with a numerical score. The rejection of other languages is explained with goal-specific reasoning. If overridden, the override is acknowledged.

---

### Step 4: Create a Plan

**Description:**  
Convert the restated goal into an ordered, phased development plan. Define milestones, file inventory, test inventory, and per-phase completion criteria.

**Agent Responsible:** Planner Agent

**Input:**
- Restated goal
- Recommended language
- Request type
- Acceptance checklist

**Output:**
- Numbered phases (3-8)
- Per-phase: name, description, files, tests, milestone, complexity
- Complete file inventory
- Dependency list with versions
- Per-phase completion criteria

**What "Done" Looks Like:**  
A developer could hand this plan to another developer and get functionally equivalent output. Every acceptance criterion appears in at least one phase. Tests are specified before implementation.

---

### Step 5: Architect the Solution

**Description:**  
Design the complete technical architecture: directory structure, module boundaries, design patterns, library choices, error handling strategy, and testing strategy. Document every significant design decision.

**Agent Responsible:** Architect Agent

**Input:**
- Development plan from Step 4
- Recommended language
- Request type
- Acceptance checklist

**Output:**
- Directory structure tree
- Per-module design (purpose, interface, dependencies)
- Design patterns used
- Library choices with versions and justifications
- Error handling strategy
- Testing strategy
- Documented design decisions

**What "Done" Looks Like:**  
The architecture is complete enough that a developer could implement it without further design decisions. Every module has clear responsibilities. Every dependency is justified.

---

### Step 6: Sandbox the Code

**Description:**  
The Language Specialist audits the architecture against language-specific conventions. The Sandbox Coding Agent then writes the first working version. Focus is on completeness and correctness, not elegance or optimization.

**Agent Responsible:** Language Specialist Agent + Sandbox Coding Agent

**Input:**
- Architecture from Step 5
- Language specialist rules for the selected language
- Development plan from Step 4
- Acceptance checklist

**Output:**
- Language audit report (pass/fail with notes)
- Complete source code for all modules
- Initial test suite
- Build configuration
- Usage examples
- Known issues list

**What "Done" Looks Like:**  
Code compiles and runs. All acceptance criteria are implemented. Tests exist for every module. Examples produce correct output. No feature is left partially implemented.

---

### Step 7: Write the First Implementation

**Description:**  
Produce the complete, polished first implementation. All files from the architecture are fully coded. All tests from the plan are written. Build config is complete and functional.

**Agent Responsible:** Sandbox Coding Agent (continued)

**Input:**
- Architecture design
- Language specialist rules
- Development plan

**Output:**
- All source files (complete, not stubs)
- All test files
- Build/run configuration
- README with basic usage
- Example code

**What "Done" Looks Like:**  
`cargo test` / `npm test` / `julia test/runtests.jl` / `pytest` / `make test` runs and shows test results. The code does what the goal describes. A new developer can clone and run.

---

### Step 8: Test the Implementation

**Description:**  
The Test Agent writes and executes a comprehensive test suite: happy paths, edge cases, invalid inputs, regressions, and performance tests. Coverage is measured and gaps are documented.

**Agent Responsible:** Test Agent

**Input:**
- Source code from Step 7
- Acceptance checklist
- Architecture design

**Output:**
- Happy path tests
- Edge case tests
- Invalid input tests
- Regression tests
- Performance tests (if applicable)
- Coverage report (% and untested lines)
- Test results (pass/fail per test)
- Documented test gaps

**What "Done" Looks Like:**  
All tests pass. Coverage is reported. Every acceptance criterion has at least one corresponding test. Edge cases include empty input, maximums, boundaries, and nulls.

**On Failure:**  
Return to Step 6. Fix the code. Re-run tests. Repeat until all tests pass.

---

### Step 9: Review Against Language-Specific Best Practices

**Description:**  
The Best Practices Review Agent scores the code on 8 dimensions: language idioms, security, maintainability, performance, simplicity, error handling, naming, and file structure. Required improvements must be fixed before proceeding.

**Agent Responsible:** Best Practices Review Agent

**Input:**
- Source code from Step 7
- Test suite from Step 8
- Language specialist rules
- Architecture design

**Output:**
- Score card (1-10 per dimension, weighted total)
- Required improvements list (blocking)
- Recommended improvements list (non-blocking)
- Positive findings

**What "Done" Looks Like:**  
Weighted total score is 7/10 or higher. All required improvements are documented with specific file/line references. Security issues are flagged as required.

**On Failure (score < 7):**  
Return to Step 6. Fix all required issues. Re-run through Steps 7-9.

---

### Step 10: Compare the Code Against the Original Goal

**Description:**  
The Goal Comparison Agent verifies the delivered code matches the original restated goal. Every acceptance criterion is checked. Scope creep and missing features are identified.

**Agent Responsible:** Goal Comparison Agent

**Input:**
- Original restated goal
- Acceptance checklist
- Source code and tests
- Architecture design

**Output:**
- Per-criterion check: Met / Partially Met / Not Met
- Scope creep report (unrequested features)
- Missing features report
- Mismatch analysis
- Proceed / Fix and Re-check recommendation

**What "Done" Looks Like:**  
Every acceptance criterion is explicitly checked. The recommendation is unambiguous. If "Fix", a specific change list is provided.

**On Mismatch:**  
Return to Step 4. Re-plan. Re-code. Re-test. Re-review.

---

### Step 11: Optimize

**Description:**  
The Optimization Agent improves the code in priority order: correctness, simplicity, safety, performance, usability. Every optimization is explained with before/after evidence. No optimization breaks tests.

**Agent Responsible:** Optimization Agent

**Input:**
- Source code (all tests passing)
- Best Practices Review score card
- Goal comparison results
- Performance benchmarks (if available)

**Output:**
- Optimization log (what, why, before/after, category)
- Updated source code
- Test verification (still passing)

**What "Done" Looks Like:**  
All tests still pass. Every optimization has a written justification. No premature or unjustified optimization was applied.

**On Test Failure:**  
Return to Optimization Agent. Revert breaking change. Find alternative optimization.

---

### Step 12: Run a Final Review

**Description:**  
The Final Review Agent checks 7 gates: goal match, test pass, best practices score, clear usage, no security issues, no unnecessary complexity, known limitations documented. All gates must pass.

**Agent Responsible:** Final Review Agent

**Input:**
- Final source code
- Final test suite and results
- Best Practices Review score card
- Goal Comparison report
- Optimization log

**Output:**
- Gate results (Pass/Fail per gate)
- Final score (1-10)
- Known limitations list
- Delivery approval or rejection

**What "Done" Looks Like:**  
All 7 gates pass. Known limitations are documented. The final score is honest.

**On Failure:**  
Return to the agent responsible for the failed gate. Fix and re-run from that step.

---

### Step 13: Present Final Code

**Description:**  
Deliver the complete package to the user: source code files, test suite, usage instructions, known limitations, and quality score.

**Agent Responsible:** Final Review Agent (delivery phase)

**Input:**
- All approved outputs from Steps 1-12

**Output:**
- Source code (all files)
- Test suite (all files)
- Build/run instructions
- Usage examples
- Known limitations document
- Final quality score
- Summary of agent decisions and scores

**What "Done" Looks Like:**  
The user has everything needed to use the code. A new developer can clone, build, test, and understand the project without additional help.

---

## Error Handling

### What Happens When a Step Fails

Each step has a defined failure mode and recovery path:

| Failing Step | Recovery Action | Re-entry Point |
|-------------|-----------------|----------------|
| Step 1 | Ask user for clarification | Step 1 |
| Step 2 | Ask user for clarification | Step 2 |
| Step 3 | User override or re-analysis | Step 3 |
| Step 4 | Re-scope with user input | Step 4 |
| Step 5 | Simplify architecture | Step 5 |
| Step 6 | Fix compilation errors | Step 6 |
| Step 7 | Fix implementation gaps | Step 6-7 |
| Step 8 | Fix code to pass tests | Step 6 |
| Step 9 | Fix required review issues | Step 6 |
| Step 10 | Re-plan missing features | Step 4 |
| Step 11 | Revert and re-optimize | Step 11 |
| Step 12 | Fix failed gate | Step matching failed gate |
| Step 13 | N/A — delivery always succeeds | — |

### Maximum Iterations

To prevent infinite loops:

| Loop Type | Maximum Iterations |
|-----------|-------------------|
| Test fix loop (Step 8 → 6) | 5 |
| Review fix loop (Step 9 → 6) | 3 |
| Goal mismatch loop (Step 10 → 4) | 2 |
| Optimization loop (Step 11 failure) | 3 |
| Final gate loop (Step 12 failure) | 2 |

After maximum iterations, the agent escalates to the user with:
- Current status
- Why the loop is happening
- Recommended resolution
- Option to accept current state with documented limitations

---

## Iteration Rules

### When to Loop Back

1. **Tests fail in Step 8** → Always loop to Step 6. Fix code. Re-test.
2. **Review score < 7 in Step 9** → Loop to Step 6. Fix required issues.
3. **Goal mismatch in Step 10** → Loop to Step 4. Re-plan and re-implement.
4. **Optimization breaks tests in Step 11** → Stay in Step 11. Revert. Try again.
5. **Final gate fails in Step 12** → Loop to the step responsible for the failed gate.

### When NOT to Loop Back

1. Review score >= 7 with only recommended (non-blocking) improvements → proceed
2. Scope creep detected but user approves → proceed with note
3. Minor optimization opportunity with negligible benefit → skip
4. Test gap in non-critical path → document and proceed

### Loop Prevention

The framework prevents infinite loops by:
- Tracking iteration count per loop type
- Requiring demonstrable progress on each iteration
- Escalating to user after max iterations
- Allowing "accept with limitations" as a resolution

---

## Workflow State Machine

```
                    ┌──────────┐
         ┌─────────│ START    │◄────────┐
         │         └────┬─────┘         │
         │              │               │
         │              ▼               │
         │      ┌───────────────┐       │
         │      │ Steps 1-5     │       │
         │      │ Plan & Design │       │
         │      └───────┬───────┘       │
         │              │               │
         │              ▼               │
         │      ┌───────────────┐       │
         │      │ Steps 6-7     │       │
         │      │ Implement     │       │
         │      └───────┬───────┘       │
         │              │               │
         │              ▼               │
         │      ┌───────────────┐       │
         │      │ Step 8        │       │
         │      │ Tests Pass?   │       │
         │      └───────┬───────┘       │
         │          NO  │  YES           │
         │         ┌────┘               │
         │         ▼                    │
         │    ┌──────────┐              │
         │    │ Fix Code │──────────────┘
         │    └──────────┘
         │
         │              ▼
         │      ┌───────────────┐
         │      │ Step 9        │
         │      │ Review >= 7?  │
         │      └───────┬───────┘
         │          NO  │  YES
         │         ┌────┘
         │         ▼
         │    ┌──────────┐
         │    │ Fix Code │───────┐
         │    └──────────┘       │
         │                       │
         │              ▼        │
         │      ┌───────────────┐│
         │      │ Step 10       ││
         │      │ Matches Goal? ││
         │      └───────┬───────┘│
         │          NO  │  YES   │
         │         ┌────┘        │
         │         ▼             │
         │    ┌──────────┐       │
         │    │ Re-plan  │───────┘
         │    └──────────┘
         │
         │              ▼
         │      ┌───────────────┐
         │      │ Step 11       │
         │      │ Optimize      │
         │      └───────┬───────┘
         │              │
         │              ▼
         │      ┌───────────────┐
         │      │ Step 12       │
         │      │ Final Gates   │
         │      └───────┬───────┘
         │          NO  │  YES
         │         ┌────┘
         │         ▼
         │    ┌──────────┐
         │    │ Fix Gate │───────┐
         │    └──────────┘       │
         │                       │
         │              ▼        │
         │      ┌───────────────┐│
         │      │ Step 13       ││
         │      │ Deliver       ││
         │      └───────────────┘│
         │                       │
         │              ▼        │
         │         ┌────────┐    │
         └────────►│ DONE   │────┘
                   └────────┘
```

---

## Parallel Execution Notes

While the framework is primarily sequential, these operations can run in parallel:

- **Step 6 (Language Specialist audit)** can happen while the Sandbox agent prepares the coding environment
- **Step 9 (Best Practices Review)** and **Step 10 (Goal Comparison)** are independent and can run in either order
- **Step 11 (Optimization)** categories can be processed in batches

In practice, Claude Code executes these sequentially. The workflow diagram shows logical dependencies, not physical execution order.

---

## Summary Table

| Step | Description | Agent | Input | Output | Done When |
|------|-------------|-------|-------|--------|-----------|
| 1 | Understand goal | Goal Analyst | User's description | Restated goal, acceptance checklist | Goal is specific and verifiable |
| 2 | Analyze requirements | Goal Analyst | Restated goal | Request type, constraints | Type classified, constraints documented |
| 3 | Select language | Query Logic | Goal + requirements | Language recommendation | One language recommended with scores |
| 4 | Create plan | Planner | Goal + language | Phased plan with milestones | Plan covers all acceptance criteria |
| 5 | Architect solution | Architect | Plan | Structure, patterns, libraries | Design is implementation-ready |
| 6 | Sandbox code | Language Specialist + Sandbox | Architecture | Working code + tests | Code compiles, all features present |
| 7 | First implementation | Sandbox | Architecture | Complete source + tests + config | Project builds and runs |
| 8 | Test implementation | Test Agent | Source code | Test suite + coverage + results | All tests pass |
| 9 | Best practices review | Review Agent | Source + tests | Score card + required fixes | Score >= 7/10 |
| 10 | Compare to goal | Goal Comparator | Goal + code | Match/mismatch report | All criteria met |
| 11 | Optimize | Optimizer | Code + scores | Optimization log + updated code | Tests still pass, changes justified |
| 12 | Final review | Final Review | Everything | Gate results + final score | All 7 gates pass |
| 13 | Present final code | Final Review | Approved outputs | Delivery package | User has everything needed |