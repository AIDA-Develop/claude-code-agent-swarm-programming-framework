# Language Decision Matrix

Quick-reference guide for selecting the optimal language. Use this for fast decisions or to understand the Query Logic Agent's recommendation.

---

## Quick Reference Matrix

### Base Scores

| Category | Rust | TypeScript | Julia | C++ | Python |
|----------|------|------------|-------|-----|--------|
| Raw performance | High | Medium | High | Very High | Low |
| Memory safety | Very High | Medium | High | Low-Medium | Medium |
| Ease of development | Medium | High | High | Low-Medium | Very High |
| AI app integration | Medium | Very High | Medium | Medium | Very High |
| Numerical computing | Medium | Medium | Very High | High | High |
| Web/frontend support | Low | Very High | Low | Low | Low |
| Backend API support | High | Very High | Medium | High | High |
| Systems programming | Very High | Low | Medium | Very High | Low |
| Concurrency | Very High | High | Medium | High | Low-Medium |
| Ecosystem for AI apps | Medium | Very High | Medium | Medium | Very High |
| Production maintainability | Very High | High | Medium | Medium | Medium |
| Hardware-level control | High | Low | Medium | Very High | Low |
| Learning ease | Low | High | Medium | Very Low | Very High |

### Point Values

| | Very High | High | Medium | Low-Medium | Low | Very Low |
|--|-----------|------|--------|------------|-----|----------|
| Points | 4 | 3 | 2 | 1.5 | 1 | 0 |

---

## Detailed Language Profiles

### Rust

**Best for:** Systems programming, performance-critical services, CLI tools, network services, embedded systems (with `no_std`), WebAssembly, anything requiring memory safety at speed.

**Strengths:**
- Memory safety without garbage collection (ownership system)
- Zero-cost abstractions
- Fearless concurrency (compile-time data race prevention)
- Excellent package manager and build tool (Cargo)
- Strong type system with algebraic data types
- Growing ecosystem with high-quality crates
- First-class WebAssembly support
- Cross-platform compilation

**Weaknesses:**
- Steep learning curve (ownership, lifetimes, borrowing)
- Longer compile times
- Smaller ecosystem than C++ or TypeScript for some domains
- String processing is more verbose than scripting languages
- Limited GUI framework maturity
- AI/ML ecosystem is nascent ( relies on ONNX, Candle, or FFI)

**Ideal Use Cases:**
- High-performance network services (APIs, proxies, load balancers)
- CLI tools (ripgrep, fd, bat are Rust)
- Systems software (operating systems, containers, virtualization)
- Embedded and bare-metal programming
- WebAssembly modules
- Blockchain and cryptocurrency infrastructure
- Game engines (Bevy) and graphics

**When NOT to use:**
- Quick prototypes where development speed matters most
- Projects requiring mature GUI frameworks
- Heavy AI/ML model training (Python/Julia domain)
- Teams with no Rust experience and no time to learn

---

### TypeScript

**Best for:** Web applications, backend APIs, AI-powered apps, full-stack development, rapid prototyping, browser extensions, mobile apps (React Native).

**Strengths:**
- Largest ecosystem (npm has 2M+ packages)
- Seamless AI integration (OpenAI SDK, Vercel AI SDK, LangChain)
- Full-stack capability (React, Next.js, Express, NestJS)
- Rapid development with excellent DX
- Strong tooling (VS Code, ESLint, Prettier)
- Massive community and hiring pool
- Easy deployment (Node.js, Deno, Bun, edge runtimes)
- Excellent for JSON-heavy APIs and data transformation

**Weaknesses:**
- Single-threaded (event loop) — CPU-bound tasks block
- Memory overhead from V8 engine
- Type system has escape hatches (`any`, `ts-ignore`)
- Runtime errors possible despite compile-time types
- Performance ceiling lower than compiled languages
- Dependency tree can become bloated

**Ideal Use Cases:**
- REST and GraphQL APIs
- React/Vue/Angular web applications
- AI chatbots and agents
- Real-time applications (WebSocket, SSE)
- Serverless and edge functions
- Internal tools and dashboards
- Browser extensions
- Mobile apps with React Native

**When NOT to use:**
- CPU-bound numerical computing
- Systems programming or OS-level code
- Embedded or resource-constrained environments
- Real-time systems with hard latency guarantees
- High-frequency trading or similar ultra-low-latency needs

---

### Julia

**Best for:** Numerical computing, scientific simulation, data analysis, machine learning research, optimization problems, differential equations, linear algebra, statistics.

**Strengths:**
- Designed for numerical computing (syntax mirrors math)
- Multiple dispatch — elegant, extensible, fast
- Near-C performance with JIT compilation (LLVM)
- Excellent for linear algebra (built-in matrix operations)
- Rich scientific ecosystem (DifferentialEquations.jl, Flux.jl, DataFrames.jl)
- Easy interoperability with C, Fortran, Python (PyCall)
- Built-in package manager and environment system
- REPL-driven development (like Python, but faster)

**Weaknesses:**
- First-time JIT compilation is slow ("time to first plot" problem)
- Smaller general-purpose ecosystem than Python/JS
- Not ideal for systems programming or web frontends
- Deployment story is less mature (no single-binary like Rust/Go)
- Garbage collection can pause execution
- Smaller hiring pool than Python or TypeScript
- IDE support less mature than Rust/TypeScript

**Ideal Use Cases:**
- Scientific computing and simulations
- Machine learning research and experimentation
- Data analysis and visualization
- Optimization and operations research
- Differential equations modeling
- Financial modeling and risk analysis
- Signal processing and image analysis
- Linear algebra-heavy computations

**When NOT to use:**
- Web frontend development
- High-concurrency network services
- Embedded or bare-metal programming
- Deployment as a small CLI tool (startup latency)
- Teams without numerical computing background

---

### C++

**Best for:** Extreme performance, hardware-level control, game engines, embedded systems, real-time systems, high-frequency trading, operating systems, legacy system integration.

**Strengths:**
- Maximum performance (zero-overhead principle)
- Direct hardware control (memory, registers, intrinsics)
- Massive ecosystem (40+ years of libraries)
- Deterministic performance (no GC, predictable latency)
- Multi-paradigm (OOP, generic, functional, procedural)
- Industry standard for game engines (Unreal, Unity core), OS (Windows, Linux), browsers (Chrome, Firefox)
- Strong for real-time and embedded systems
- CUDA and GPU programming

**Weaknesses:**
- Manual memory management (error-prone, security risks)
- Complex syntax and steep learning curve
- Long compile times
- Header/implementation separation is tedious
- Dependency management is notoriously difficult
- Undefined behavior is easy to trigger
- Debugging memory issues requires specialized tools (Valgrind, ASan)
- Slower development velocity than Rust or TypeScript

**Ideal Use Cases:**
- Game engines and real-time graphics
- Operating systems and kernels
- High-frequency trading systems
- Embedded and bare-metal firmware
- Audio/video processing codecs
- Database engines
- Browser engines
- Physics engines and simulations
- GPU computing (CUDA, OpenCL)
- Legacy system maintenance

**When NOT to use:**
- Web applications or APIs
- Rapid prototyping
- Teams without C++ expertise
- Projects where memory safety is critical and Rust is an option
- AI application development (use Python/TypeScript)

---

### Python

**Best for:** AI/ML integration, automation and scripting, data pipelines, rapid prototyping, internal backend services, and glue code that ties systems together — anywhere iteration speed and library availability matter more than raw throughput.

**Strengths:**
- Largest AI/ML ecosystem (PyTorch, TensorFlow, the model-provider SDKs all land here first)
- Lowest learning curve of the five — fastest from idea to working code
- Vast general-purpose library ecosystem (PyPI)
- Excellent data tooling (numpy, pandas, scipy)
- Strong, typed web frameworks (FastAPI) and rich CLI/automation support
- Type hints + mypy bring optional static safety where you want it
- Ubiquitous — largest hiring pool, runs everywhere

**Weaknesses:**
- Runtime performance is the slowest of the five (interpreted)
- The GIL serializes CPU-bound threads (parallel CPU work needs multiprocessing)
- Dynamic typing is a maintainability risk at scale without type-hint discipline
- No bare-metal / embedded story; requires an interpreter and runtime
- Startup latency makes it weak for tiny short-lived CLI invocations
- Packaging/deployment is less clean than a single compiled binary

**Ideal Use Cases:**
- AI/ML application integration and inference glue
- Automation, scripting, and orchestration
- Data pipelines, ETL, and analysis
- Rapid prototyping and internal tools
- Backend APIs where iteration speed beats raw throughput (FastAPI/Django)
- Scientific computing and notebooks
- ML experimentation and model training

**When NOT to use:**
- Hard real-time or bare-metal/embedded systems
- Latency-critical hot paths or high-throughput network services
- CPU-bound parallel workloads (GIL)
- Single-binary CLI tools where startup time matters

## Decision Flowchart (ASCII)

```
                        START
                          │
                          ▼
                ┌─────────────────┐
                │ Is it AI/ML      │
                │ integration,     │
                │ automation, or a │
                │ data pipeline?   │
                └────────┬────────┘
                   YES   │   NO
                   ┌─────┘
                   ▼
            ┌──────────┐
            │  Python  │  (when the surrounding system / model SDKs are Python;
            └──────────┘   use TypeScript instead if it must ship in a web app)
                          │
                          ▼
                ┌─────────────────┐
                │ Does it need    │
                │ a web UI or     │
                │ browser?        │
                └────────┬────────┘
                   YES   │   NO
                   ┌─────┘
                   ▼
            ┌────────────┐
            │ TypeScript │◄─────────────────────────┐
            └────────────┘                          │
                                                    │
                          ▼                         │
                ┌─────────────────┐                 │
                │ Does it involve │                 │
                │ heavy numerical │                 │
                │ computing?      │                 │
                └────────┬────────┘                 │
                   YES   │   NO                     │
                   ┌─────┘                          │
                   ▼                                │
            ┌──────────────────┐                    │
            │ Does it need to  │                    │
            │ deploy as a web  │                    │
            │ service or API?  │                    │
            └────────┬─────────┘                    │
               YES   │   NO                         │
               ┌─────┘                              │
               ▼                                    │
        ┌────────────┐    ┌──────────┐              │
        │ TypeScript │    │  Julia   │              │
        └────────────┘    └──────────┘              │
                                                    │
                          ▼                         │
                ┌─────────────────┐                 │
                │ Does it need    │                 │
                │ hardware-level  │                 │
                │ control or      │                 │
                │ bare metal?     │                 │
                └────────┬────────┘                 │
                   YES   │   NO                     │
                   ┌─────┘                          │
                   ▼                                │
            ┌──────────────────┐                    │
            │ Is memory safety │                    │
            │ a hard           │                    │
            │ requirement?     │                    │
            └────────┬─────────┘                    │
               YES   │   NO                         │
               ┌─────┘                              │
               ▼                                    │
        ┌──────────┐      ┌──────────┐              │
        │   Rust   │      │   C++    │              │
        └──────────┘      └──────────┘              │
                                                    │
                          ▼                         │
                ┌─────────────────┐                 │
                │ Is performance  │                 │
                │ critical and    │                 │
                │ safety matters? │                 │
                └────────┬────────┘                 │
                   YES   │   NO                     │
                   ┌─────┘                          │
                   ▼                                │
            ┌──────────┐      ┌──────────────┐      │
            │   Rust   │      │ Does it need │      │
            └──────────┘      │ AI/ML API    │      │
                              │ integration? │      │
                              └──────┬───────┘      │
                                 YES │   NO         │
                                 ┌───┘              │
                                 ▼                  │
                          ┌────────────┐            │
                          │ TypeScript │────────────┘
                          └────────────┘
                                                    │
                          ▼                         │
                ┌─────────────────┐                 │
                │ Does it involve │                 │
                │ systems         │                 │
                │ programming?    │                 │
                └────────┬────────┘                 │
                   YES   │   NO                     │
                   ┌─────┘                          │
                   ▼                                │
            ┌──────────┐      ┌──────────────┐      │
            │   Rust   │      │ General CLI  │      │
            └──────────┘      │ or tool?     │      │
                              └──────┬───────┘      │
                                 YES │   NO         │
                                 ┌───┘              │
                                 ▼                  │
                          ┌──────────┐              │
                          │   Rust   │              │
                          └──────────┘              │
                                                    │
                          ▼                         │
                ┌─────────────────┐                 │
                │ Team knows      │                 │
                │ TypeScript best │                 │
                │ and speed matters│                │
                └────────┬────────┘                 │
                   YES   │   NO                     │
                   ┌─────┘                          │
                   ▼                                │
            ┌────────────┐    ┌──────────┐          │
            │ TypeScript │    │   Rust   │          │
            └────────────┘    └──────────┘          │
```

---

## Common Scenario Recommendations

### Web Development

| Scenario | Recommended | Runner-up | Why |
|----------|-------------|-----------|-----|
| React/Next.js frontend | TypeScript | — | Native browser support |
| REST API server | TypeScript | Rust | Ecosystem, DX |
| GraphQL API | TypeScript | Rust | Apollo, Prisma ecosystems |
| Real-time chat (WebSocket) | TypeScript | Rust | Socket.io, event-driven model |
| AI-powered web app | TypeScript | — | AI SDKs, deployment ease |

### Systems & Infrastructure

| Scenario | Recommended | Runner-up | Why |
|----------|-------------|-----------|-----|
| CLI tool | Rust | TypeScript | Single binary, fast, safe |
| Container runtime | Rust | C++ | Memory safety at systems level |
| Network proxy/load balancer | Rust | C++ | Tokio async, zero-copy |
| Kernel module | C++ / Rust | C | Hardware access |
| File system | Rust | C++ | Safety-critical, performance |

### Data & AI

| Scenario | Recommended | Runner-up | Why |
|----------|-------------|-----------|-----|
| Data analysis pipeline | Python | Julia | pandas/numpy ecosystem, iteration speed |
| ML model training | Python | Julia | PyTorch/TensorFlow dominate the ecosystem |
| ML model serving | TypeScript | Python | ONNX Runtime + deployment; Python when SDKs are Python |
| Statistical modeling | Julia | Python | Built-in statistics; Python (statsmodels) a close second |
| Big data ETL | Rust | Python | Memory efficiency vs. development speed |
| AI-glue / automation service | Python | TypeScript | Model SDKs + libraries land in Python first |

### Embedded & Hardware

| Scenario | Recommended | Runner-up | Why |
|----------|-------------|-----------|-----|
| Bare-metal ARM | Rust | C++ | `no_std`, memory safety |
| Arduino-class device | C++ | — | Ecosystem maturity |
| Real-time control system | C++ | Rust | Deterministic latency |
| FPGA soft-core | Rust / C++ | — | Hardware description |
| IoT sensor firmware | Rust | C++ | Safety, power efficiency |

### Games & Graphics

| Scenario | Recommended | Runner-up | Why |
|----------|-------------|-----------|-----|
| Game engine | C++ | Rust | Unreal/Unity are C++ |
| 2D game (simple) | TypeScript | Rust | Browser deployment |
| Shader/compute | C++ | Rust | GPU API bindings |
| Ray tracer | Rust / C++ | Julia | Performance, parallelism |

---

## Override Rules

### When to Override the Default Recommendation

| Override Reason | When Applied | Example |
|-----------------|-------------|---------|
| **Team expertise** | Team has deep skills in a lower-scoring language | "Our team of 10 C++ engineers can build this safer than learning Rust from scratch" |
| **Existing codebase** | Must integrate with existing code | "This must integrate with our TypeScript monorepo" |
| **Deployment constraint** | Target platform restricts languages | "Target is AWS Lambda — TypeScript has best cold start" |
| **Library dependency** | Critical library exists in only one language | "Must use PyTorch C++ API directly" |
| **Organizational policy** | Company/team mandates a language | "All backend services must be TypeScript per CTO directive" |
| **Time constraint** | Hard deadline requires known tools | "Demo is in 3 days, team only knows TypeScript" |
| **Hiring pool** | Need to hire for the project | "Need to hire 5 devs; TypeScript has largest pool" |

### Override Process

1. State the override reason clearly
2. Acknowledge what you're sacrificing
3. Document risk mitigation steps
4. Get explicit approval (from team lead, architect, or user)

### Override Template

```markdown
## Language Override

Original recommendation: [language] (score: [X])
Overridden to: [language] (score: [Y])

### Reason
[One clear sentence]

### What We Gain
- [Benefit 1]
- [Benefit 2]

### What We Sacrifice
- [Sacrifice 1]
- [Sacrifice 2]

### Risk Mitigation
- [How we'll address the risks of the override]

### Approval
Approved by: [name/role]
Date: [date]
```

---

## Team and Organizational Factors

### Team Size

| Team Size | Recommendation |
|-----------|---------------|
| Solo developer | Use what you know best; framework helps with consistency |
| Small (2-5) | Consensus-driven; framework provides objective scores |
| Medium (5-15) | Default to highest-scoring language; override for team cohesion |
| Large (15+) | Strongly prefer highest-scoring language; standardization matters |

### Experience Level

| Experience | Recommendation |
|------------|---------------|
| All senior | Can handle Rust or C++ effectively |
| Mixed | TypeScript, Python, or Julia for faster onboarding |
| All junior | TypeScript or Python for ecosystem support and hiring |
| Learning-focused | Use the project to learn the highest-scoring language |

### Project Duration

| Duration | Recommendation |
|----------|---------------|
| < 1 week prototype | Fastest language team knows |
| 1-4 weeks | Framework's highest scorer |
| 1-6 months | Highest scorer with team skill consideration |
| 6+ months | Highest scorer; invest in team learning if needed |
| Indefinite (product) | Highest scorer; production maintainability is critical |

### Maintenance Expectancy

| Expectancy | Recommendation |
|------------|---------------|
| Throwaway (< 1 month) | Fastest to write |
| Short-term (1-6 months) | Team's strongest language |
| Medium-term (6-24 months) | Framework's highest scorer |
| Long-term (2+ years) | Rust or TypeScript for maintainability |
| Permanent product | Rust for safety; TypeScript for web; Julia for science |

---

## Summary: The 60-Second Decision

1. **AI/ML integration, automation, data pipelines, or glue code?** → Python
2. **Web/UI/API?** → TypeScript
3. **Numerical computing/science?** → Julia
4. **Hardware/bare metal/extreme performance?** → Rust (if safety matters) or C++ (if team knows it)
5. **Systems programming/CLI/network?** → Rust
6. **Unclear?** → Run the full scoring in [QUERY_LOGIC.md](QUERY_LOGIC.md)
7. **Team/org constraint?** → Override with documented rationale

When in doubt, the framework defaults to the language that scores highest on the weighted matrix. The matrix is objective; override decisions are documented and auditable.