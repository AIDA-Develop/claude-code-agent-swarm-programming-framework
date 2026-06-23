# Query Analysis Framework

How the Query Logic Agent evaluates goals and selects the optimal programming language.

---

## Why Query Analysis Matters

Language selection is the single most consequential decision in the workflow. The wrong language creates friction for the entire project: slower development, harder maintenance, security issues, or deployment problems.

The Query Logic Agent makes this decision systematically using a scored matrix rather than intuition. Every recommendation is explainable and auditable.

---

## The Scoring Table

Base scores for each language across 14 categories. Scores are on a qualitative scale:

- **Very High** = 4 points
- **High** = 3 points
- **Medium** = 2 points
- **Low-Medium** = 1.5 points
- **Low** = 1 point
- **Very Low** = 0 points

Every category is scored so that **higher is better for the project**. (Note that
"Learning ease" means *ease* of learning — higher = easier to pick up — and
"Security risk" is scored as safety — higher = safer.)

```
Category                         Rust   TypeScript   Julia   C++          Python
-----------------------------------------------------------------------------------
Raw performance                  High   Medium       High    Very High    Low
Memory safety                    Very   High         Medium  Low-Medium   Medium
                                 High
Ease of development              Medium High         High    Low-Medium   Very High
AI app integration               Medium Very High    Medium  Medium       Very High
Numerical computing              Medium Medium       Very    High         High
                                                          High
Web/frontend support             Low    Very High    Low     Low          Low
Backend API support              High   Very High    Medium  High         High
Systems programming              Very   Low          Medium  Very High    Low
                                 High
Concurrency                      Very   High         Medium  High         Low-Medium
                                 High
Ecosystem for AI apps            Medium Very High    Medium  Medium       Very High
Production maintainability       Very   High         Medium  Medium       Medium
                                 High
Hardware-level control           High   Low          Medium  Very High    Low
Learning ease                    Low    High         Medium  Very Low     Very High
```

### Numerical Mapping

| Qualitative | Points |
|-------------|--------|
| Very High | 4 |
| High | 3 |
| Medium | 2 |
| Low-Medium | 1.5 |
| Low | 1 |
| Very Low | 0 |

---

## Weighted Scoring Methodology

Not all categories matter equally for every project. The Query Logic Agent applies weights based on the specific goal.

### Step 1: Identify Relevant Categories

From the restated goal and request type, the agent identifies which categories are:
- **Critical** (weight 3x) — the project's primary requirement
- **Important** (weight 2x) — secondary requirements
- **Standard** (weight 1x) — relevant but not differentiating
- **Irrelevant** (weight 0x) — not applicable to this project

### Step 2: Apply Weights

Multiply each language's base score by the category weight.

### Step 3: Sum Weighted Scores

Total the weighted scores per language. The highest score wins.

### Step 4: Apply Tiebreakers

If scores are within 10% of each other:
1. Prefer the language with the higher score on critical-weighted categories
2. Prefer the language with better production maintainability
3. Prefer the language with higher learning ease (team velocity)
4. If still tied, recommend the language the user is most likely to know

### Step 5: Document Rejection

For each non-recommended language, write one sentence explaining why it lost, specific to this goal.

---

## Decision Algorithm

```
FUNCTION SelectLanguage(goal, requestType, requirements):

    // Step 1: Map qualitative scores to numerical
    scores = {
        Rust:        [3, 4, 2, 2, 2, 1, 3, 4, 4, 2, 4, 3, 1, 3],
        TypeScript:  [2, 2, 3, 4, 2, 4, 4, 1, 3, 4, 3, 1, 3, 2],
        Julia:       [3, 2, 3, 2, 4, 1, 2, 2, 2, 2, 2, 2, 2, 1],
        C++:         [4, 1, 1, 2, 3, 1, 3, 4, 3, 2, 2, 4, 0, 2],
        Python:      [1, 2, 4, 4, 3, 1, 3, 1, 1.5, 4, 2, 1, 4, 2]
    }

    categories = [
        "Raw performance", "Memory safety", "Ease of development",
        "AI app integration", "Numerical computing", "Web/frontend support",
        "Backend API support", "Systems programming", "Concurrency",
        "Ecosystem for AI apps", "Production maintainability",
        "Hardware-level control", "Learning ease", "Security risk"
    ]

    // Step 2: Assign weights based on goal
    weights = AssignWeights(goal, requestType, requirements)

    // Step 3: Calculate weighted scores
    totals = {}
    FOR each language IN [Rust, TypeScript, Julia, C++, Python]:
        totals[language] = 0
        FOR i = 0 to 13:
            totals[language] += scores[language][i] * weights[i]

    // Step 4: Find winner
    winner = language with MAX(totals)
    margin = totals[winner] - second_best(totals)

    // Step 5: Confidence level
    IF margin > 20%:
        confidence = "High"
    ELSE IF margin > 10%:
        confidence = "Medium"
    ELSE:
        confidence = "Low"
        winner = ApplyTiebreakers(winner, totals, goal)

    // Step 6: Generate rejection rationales
    rejections = {}
    FOR each language IN [Rust, TypeScript, Julia, C++, Python]:
        IF language != winner:
            rejections[language] = ExplainRejection(language, winner, goal)

    RETURN {
        recommendation: winner,
        confidence: confidence,
        scores: totals,
        matrix: scores,
        weights: weights,
        rejections: rejections
    }
```

---

## Worked Examples

### Example 1: High-Performance Concurrent Web API

**Goal:** "Build a backend API that handles 100k concurrent WebSocket connections with sub-millisecond latency. Must be memory-safe and deployable as a single binary."

**Analysis:**
- Request type: Backend Service
- Critical: Concurrency, Raw performance, Memory safety
- Important: Production maintainability, Systems programming
- Standard: Ease of development, Backend API support
- Irrelevant: Web/frontend support, Numerical computing, AI app integration

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| Raw performance | 3x | 3*3=9 | 2*3=6 | 3*3=9 | 4*3=12 | 1*3=3 |
| Memory safety | 3x | 4*3=12 | 2*3=6 | 2*3=6 | 1*3=3 | 2*3=6 |
| Concurrency | 3x | 4*3=12 | 3*3=9 | 2*3=6 | 3*3=9 | 1.5*3=4.5 |
| Systems programming | 2x | 4*2=8 | 1*2=2 | 2*2=4 | 4*2=8 | 1*2=2 |
| Production maintainability | 2x | 4*2=8 | 3*2=6 | 2*2=4 | 2*2=4 | 2*2=4 |
| Backend API support | 1x | 3*1=3 | 4*1=4 | 2*1=2 | 3*1=3 | 3*1=3 |
| Ease of development | 1x | 2*1=2 | 3*1=3 | 3*1=3 | 1*1=1 | 4*1=4 |
| Hardware-level control | 1x | 3*1=3 | 1*1=1 | 2*1=2 | 4*1=4 | 1*1=1 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** Rust: 57, C++: 44, TypeScript: 37, Julia: 36, Python: 27.5

**Recommendation:** Rust (High confidence)

**Rejections:**
- TypeScript: GC pauses and single-threaded event loop cannot guarantee sub-millisecond latency at 100k connections
- Julia: JIT compilation and GC make it unsuitable for low-latency network services
- C++: Lacks memory safety guarantees; manual memory management at this scale is error-prone
- Python: GIL and interpreter overhead cannot meet sub-millisecond latency at 100k concurrent connections

---

### Example 2: Data Analysis Dashboard with ML

**Goal:** "Create a web dashboard that loads sales data, runs anomaly detection with a pre-trained PyTorch model, and displays interactive charts. Deploy as a web service."

**Analysis:**
- Request type: Web App + AI/ML Inference
- Critical: AI app integration, Ecosystem for AI apps, Web/frontend support
- Important: Backend API support, Ease of development
- Standard: Numerical computing, Production maintainability
- Irrelevant: Hardware-level control, Systems programming, Raw performance

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| AI app integration | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Ecosystem for AI apps | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Web/frontend support | 3x | 1*3=3 | 4*3=12 | 1*3=3 | 1*3=3 | 1*3=3 |
| Backend API support | 2x | 3*2=6 | 4*2=8 | 2*2=4 | 3*2=6 | 3*2=6 |
| Ease of development | 2x | 2*2=4 | 3*2=6 | 3*2=6 | 1*2=2 | 4*2=8 |
| Numerical computing | 1x | 2*1=2 | 2*1=2 | 4*1=4 | 3*1=3 | 3*1=3 |
| Production maintainability | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 2*1=2 | 2*1=2 |
| Memory safety | 1x | 4*1=4 | 2*1=2 | 2*1=2 | 1*1=1 | 2*1=2 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** TypeScript: 57, Python: 48, Rust: 35, Julia: 33, C++: 29

**Recommendation:** TypeScript (High confidence)

**Rejections:**
- Rust: Web/frontend ecosystem is immature; AI/ML integration requires FFI bindings
- Julia: No mature web frontend framework; web deployment ecosystem is limited
- C++: No built-in web framework; AI integration requires complex binding code
- Python: Strong on the ML half and a clear runner-up, but has no native frontend story; the interactive charts would need a separate JS/TS layer

---

### Example 3: Scientific Computing — ODE Solver

**Goal:** "Implement an adaptive Runge-Kutta solver for stiff ODE systems. Must support sparse Jacobian matrices and produce publication-quality plots. Performance matters but correctness is paramount."

**Analysis:**
- Request type: Numerical Computing
- Critical: Numerical computing, Ease of development
- Important: Raw performance, Production maintainability
- Standard: Memory safety
- Irrelevant: Web/frontend support, AI app integration, Backend API support, Systems programming, Hardware-level control

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| Numerical computing | 3x | 2*3=6 | 2*3=6 | 4*3=12 | 3*3=9 | 3*3=9 |
| Ease of development | 3x | 2*3=6 | 3*3=9 | 3*3=9 | 1*3=3 | 4*3=12 |
| Raw performance | 2x | 3*2=6 | 2*2=4 | 3*2=6 | 4*2=8 | 1*2=2 |
| Production maintainability | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 2*1=2 | 2*1=2 |
| Memory safety | 1x | 4*1=4 | 2*1=2 | 2*1=2 | 1*1=1 | 2*1=2 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** Julia: 31, Python: 27, Rust: 26, TypeScript: 24, C++: 23

**Recommendation:** Julia (Medium confidence)

**Rejections:**
- TypeScript: Not designed for numerical computing; would require calling out to C/Fortran
- C++: Correctness is harder to guarantee; development speed is slower
- Rust: Numerical ecosystem is growing but not as mature as Julia's
- Python: numpy/scipy can do it and ease-of-development scores high, but Julia's multiple dispatch and JIT give better performance on stiff systems with sparse Jacobians

---

### Example 4: Embedded Firmware — Motor Controller

**Goal:** "Write firmware for an STM32F4 ARM Cortex-M4 microcontroller that controls a brushless DC motor via PWM. Must handle real-time constraints with <100us response time. No OS, bare metal."

**Analysis:**
- Request type: Embedded
- Critical: Hardware-level control, Raw performance, Memory safety
- Important: Systems programming, Concurrency
- Standard: Production maintainability
- Irrelevant: Web/frontend, AI app integration, Backend API support, Numerical computing, Ease of development, Learning ease

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| Hardware-level control | 3x | 3*3=9 | 1*3=3 | 2*3=6 | 4*3=12 | 1*3=3 |
| Raw performance | 3x | 3*3=9 | 2*3=6 | 3*3=9 | 4*3=12 | 1*3=3 |
| Memory safety | 3x | 4*3=12 | 2*3=6 | 2*3=6 | 1*3=3 | 2*3=6 |
| Systems programming | 2x | 4*2=8 | 1*2=2 | 2*2=4 | 4*2=8 | 1*2=2 |
| Concurrency | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 3*1=3 | 1.5*1=1.5 |
| Production maintainability | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 2*1=2 | 2*1=2 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** Rust: 46, C++: 40, Julia: 29, TypeScript: 23, Python: 17.5

**Recommendation:** Rust (Medium confidence)

**Rejections:**
- TypeScript: Cannot run on bare metal; requires runtime and GC
- Julia: JIT compilation and GC make it unsuitable for hard real-time constraints
- C++: Strong contender but manual memory safety is risky for safety-critical motor control
- Python: Cannot run on bare metal; requires an interpreter and runtime

**Note:** If the team has no Rust embedded experience, C++ may be the practical choice despite the lower score. See override rules in [LANGUAGE_DECISION_MATRIX.md](LANGUAGE_DECISION_MATRIX.md).

---

### Example 5: AI Inference Microservice

**Goal:** "Deploy a HuggingFace transformer model as a gRPC microservice. Must handle 1000 req/s with <50ms p99 latency. Run in a Kubernetes cluster."

**Analysis:**
- Request type: AI/ML Inference + Backend Service
- Critical: AI app integration, Ecosystem for AI apps, Backend API support
- Important: Concurrency, Raw performance, Production maintainability
- Standard: Memory safety, Ease of development
- Irrelevant: Web/frontend support, Numerical computing (inference, not training), Hardware-level control

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| AI app integration | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Ecosystem for AI apps | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Backend API support | 3x | 3*3=9 | 4*3=12 | 2*3=6 | 3*3=9 | 3*3=9 |
| Concurrency | 2x | 4*2=8 | 3*2=6 | 2*2=4 | 3*2=6 | 1.5*2=3 |
| Raw performance | 2x | 3*2=6 | 2*2=4 | 3*2=6 | 4*2=8 | 1*2=2 |
| Production maintainability | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 2*1=2 | 2*1=2 |
| Memory safety | 1x | 4*1=4 | 2*1=2 | 2*1=2 | 1*1=1 | 2*1=2 |
| Ease of development | 1x | 2*1=2 | 3*1=3 | 3*1=3 | 1*1=1 | 4*1=4 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** TypeScript: 54, Python: 46, Rust: 45, C++: 39, Julia: 35

**Recommendation:** TypeScript (Medium confidence)

**Rejections:**
- Rust: HuggingFace ecosystem is primarily Python/JS; would need ONNX runtime bindings
- C++: gRPC + ML inference is possible but development velocity is slower
- Julia: Limited gRPC and containerization ecosystem; inference-focused deployment is immature
- Python: A very close runner-up — the natural language for the model layer; loses here only on concurrency/latency for the gRPC hot path, and is a valid override when the model SDKs are Python (see override rules)

---

### Example 6: CLI Tool with String Processing

**Goal:** "Build a CLI tool that parses large log files (10GB+), extracts structured data with regex, and outputs aggregated statistics. Must be cross-platform (Linux, macOS, Windows)."

**Analysis:**
- Request type: CLI Tool
- Critical: Raw performance, Memory safety, Production maintainability
- Important: Ease of development, Systems programming
- Standard: Concurrency
- Irrelevant: Web/frontend, AI app integration, Numerical computing, Hardware-level control

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| Raw performance | 3x | 3*3=9 | 2*3=6 | 3*3=9 | 4*3=12 | 1*3=3 |
| Memory safety | 3x | 4*3=12 | 2*3=6 | 2*3=6 | 1*3=3 | 2*3=6 |
| Production maintainability | 2x | 4*2=8 | 3*2=6 | 2*2=4 | 2*2=4 | 2*2=4 |
| Ease of development | 2x | 2*2=4 | 3*2=6 | 3*2=6 | 1*2=2 | 4*2=8 |
| Systems programming | 1x | 4*1=4 | 1*1=1 | 2*1=2 | 4*1=4 | 1*1=1 |
| Concurrency | 1x | 4*1=4 | 3*1=3 | 2*1=2 | 3*1=3 | 1.5*1=1.5 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** Rust: 41, Julia: 29, TypeScript: 28, C++: 28, Python: 23.5

**Recommendation:** Rust (High confidence)

**Rejections:**
- TypeScript: Node.js memory limits and slower string processing for 10GB+ files
- Julia: Not designed for CLI text processing; startup time is slow
- C++: String processing is error-prone without memory safety; cross-platform build is harder
- Python: Workable and easy to write, but interpreter speed and startup make it slower than Rust for 10GB+ files

---

### Example 7: Local AI-Glue Automation Service

**Goal:** "Build a local service that ties together several model APIs and a vector store, exposes a small HTTP endpoint, persists events to SQLite, and is easy to extend with new modules week over week. Correctness and iteration speed matter more than raw throughput."

**Analysis:**
- Request type: AI/ML Integration + Backend Service (internal)
- Critical: AI app integration, Ecosystem for AI apps, Ease of development
- Important: Backend API support, Production maintainability
- Standard: Memory safety
- Irrelevant: Web/frontend support, Systems programming, Hardware-level control, Raw performance

| Category | Weight | Rust | TypeScript | Julia | C++ | Python |
|----------|--------|------|------------|-------|-----|--------|
| AI app integration | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Ecosystem for AI apps | 3x | 2*3=6 | 4*3=12 | 2*3=6 | 2*3=6 | 4*3=12 |
| Ease of development | 3x | 2*3=6 | 3*3=9 | 3*3=9 | 1*3=3 | 4*3=12 |
| Backend API support | 2x | 3*2=6 | 4*2=8 | 2*2=4 | 3*2=6 | 3*2=6 |
| Production maintainability | 2x | 4*2=8 | 3*2=6 | 2*2=4 | 2*2=4 | 2*2=4 |
| Memory safety | 1x | 4*1=4 | 2*1=2 | 2*1=2 | 1*1=1 | 2*1=2 |
| (others: 0x) | 0 | — | — | — | — | — |

**Totals:** TypeScript: 49, Python: 48, Rust: 36, Julia: 31, C++: 26

**Recommendation:** TypeScript and Python are within 10% — apply tiebreakers. Python ties TypeScript on the two critical AI categories and beats it on ease of development; the tiebreaker favors **Python (Low confidence over TypeScript)** when the surrounding system and model SDKs are Python. Choose TypeScript instead if the service must share a codebase with a TypeScript frontend.

**Rejections:**
- Rust: AI/ML SDK ecosystem is thin; iteration speed is slower for week-over-week extension
- Julia: Smaller library ecosystem for general API/service glue
- C++: Development velocity is poor for this kind of integration-heavy, fast-changing service

---

## Tradeoff Explanation Template

Every language recommendation must include this tradeoff section:

```
## Recommendation: [LANGUAGE]
Confidence: [High/Medium/Low]

### Scores
| Language | Weighted Total | Margin |
|----------|---------------|--------|
| Winner   | [score]       | —      |
| 2nd      | [score]       | [diff] |
| 3rd      | [score]       | [diff] |
| 4th      | [score]       | [diff] |
| 5th      | [score]       | [diff] |

### What You Gain
[3-5 bullet points: what this language gives you for this specific goal]

### What You Sacrifice
[2-4 bullet points: what you give up by not choosing the runner-up]

### When to Override
[Specific conditions where the runner-up might be better]
```

---

## Quick Reference: Category Definitions

| Category | Definition | Key Question |
|----------|-----------|--------------|
| Raw performance | Single-threaded execution speed | "Does every microsecond matter?" |
| Memory safety | Prevention of memory corruption bugs | "Can I afford segfaults or security vulnerabilities?" |
| Ease of development | Speed from idea to working code | "Is time-to-market critical?" |
| AI app integration | Connecting to AI/ML APIs and models | "Does this call OpenAI, HuggingFace, or similar?" |
| Numerical computing | Mathematical and scientific computation | "Does this involve linear algebra, ODEs, or statistics?" |
| Web/frontend support | Building user interfaces in browsers | "Does this need a visual UI?" |
| Backend API support | Building HTTP/gRPC services | "Is this a server that handles requests?" |
| Systems programming | Low-level OS-adjacent code | "Does this interact with hardware or OS directly?" |
| Concurrency | Parallel and concurrent execution | "Does this handle many simultaneous tasks?" |
| Ecosystem for AI apps | Libraries and tools for AI-powered apps | "Are there mature libraries for what I'm building?" |
| Production maintainability | Long-term code health and team scaling | "Will this be maintained for years?" |
| Hardware-level control | Direct memory, register, or device access | "Do I need to control hardware directly?" |
| Learning ease | How quickly a new developer becomes productive (higher = easier) | "Can the team pick this up fast?" |
| Security risk | Resistance to security vulnerabilities (higher = safer) | "Is this exposed to untrusted input or networks?" |

---

## Override Rules

The scoring framework recommends. It does not mandate. Override when:

1. **Team expertise** — The team has deep experience in a lower-scoring language
2. **Existing codebase** — The project must integrate with an existing codebase
3. **Deployment constraint** — The target platform only supports specific languages
4. **Library dependency** — A critical library only exists in one language
5. **Organizational policy** — The organization mandates a specific language

When overriding, document:
- The override reason
- What you're gaining
- What you're sacrificing
- Risk mitigation steps

See [LANGUAGE_DECISION_MATRIX.md](LANGUAGE_DECISION_MATRIX.md) for the full override framework.
