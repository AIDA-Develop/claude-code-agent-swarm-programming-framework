# Example User Queries

> **Purpose:** This document demonstrates how the framework handles different request types. Each example shows the complete decision pipeline from query to language recommendation and architecture sketch.
>
> **Usage:** Reference these examples when the Query Analysis Agent encounters similar requests.

---

## Example 1: High-Performance CLI Tool for Log Processing

### User Query

> "Build a high-performance CLI tool for processing large log files. It should parse multi-gigabyte files with regex pattern matching, filter by timestamp range, and output aggregated statistics (request counts per endpoint, error rate, average response time). Must handle 10GB+ files without running out of memory."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Stream-processing CLI tool for log analysis |
| **Core requirements** | Streaming I/O (no full file load), regex matching, timestamp parsing, aggregation, statistics output |
| **Hidden requirements** | Progress reporting for long-running jobs; graceful handling of malformed log lines; configurable output format (JSON/CSV/text); potentially parallel processing across log chunks |
| **Performance sensitivity** | **Critical** — must process 10GB+ with constant memory |
| **Deployment target** | Native binary, cross-platform (Linux, macOS, potentially Windows) |
| **I/O pattern** | Sequential file read (streaming), minimal allocation |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Streaming performance (zero-copy) | **10** | 4 | 5 | 9 | 25% |
| Memory safety without GC | **10** | 4 | 3 | 5 | 20% |
| Regex performance | **9** | 6 | 5 | 8 | 15% |
| CLI argument parsing ecosystem | **9** | 6 | 4 | 7 | 10% |
| Cross-platform binary deployment | **9** | 5 | 4 | 7 | 10% |
| Development velocity for this task | 7 | 7 | 6 | 5 | 10% |
| String/text processing ergonomics | **8** | 7 | 6 | 5 | 10% |
| **Weighted Score** | **9.1** | 5.3 | 4.5 | 6.8 | 100% |

### Language Recommendation

**Rust** — Unanimous choice. Zero-allocation streaming via `BufRead` lines, `regex` crate with compiled regexes, `clap` for CLI, `serde` for JSON output. Ownership system guarantees memory safety without GC pauses. `cargo build --release` produces a single portable binary.

### Architecture Sketch

```
[CLI Args] ──► clap parser
                  │
                  ▼
[File Opener] ──► memmap or BufReader (streaming)
                  │
                  ▼
[Line Parser] ──► regex captures (timestamp, endpoint, status, duration)
                  │
                  ▼
[Aggregator] ──► HashMap<endpoint, Stats> (running totals, not storing lines)
                  │
                  ▼
[Output Formatter] ──► JSON / CSV / table (serde_json, csv crate)
```

**Key crates:** `clap`, `regex`, `chrono`, `serde_json`, `csv`, `anyhow`

---

## Example 2: Web Dashboard for ML Model Visualization

### User Query

> "Create a web dashboard for visualizing ML model outputs. Users upload model predictions (CSV/JSON), and the dashboard shows interactive charts (confusion matrix, ROC curves, precision-recall, feature importance), distribution plots, and a data table with filtering. Should run in the browser with a backend for file processing."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Full-stack web application with data visualization |
| **Core requirements** | File upload, CSV/JSON parsing, statistical computation, interactive charts, filtering, table display |
| **Hidden requirements** | Client-side interactivity (zoom, pan, brush selection); responsive design; potentially real-time updates for streaming predictions; export of visualizations |
| **Performance sensitivity** | Moderate — client-side rendering must be smooth; backend handles moderate file sizes (MBs, not GBs) |
| **Deployment target** | Web server + browser |
| **Ecosystem needs** | Rich visualization libraries, HTTP server, file handling |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Web framework ecosystem | 4 | **10** | 3 | 2 | 25% |
| Visualization library availability | 3 | **10** | 4 | 2 | 25% |
| Full-stack type safety | 6 | **9** | 2 | 1 | 15% |
| File upload / HTTP handling | 5 | **9** | 3 | 3 | 10% |
| CSV/JSON parsing ergonomics | 6 | **9** | 7 | 4 | 10% |
| Development velocity | 5 | **9** | 6 | 3 | 10% |
| Browser runtime (required) | N/A | **10** | N/A | N/A | 5% |
| **Weighted Score** | 4.6 | **9.6** | 3.7 | 2.3 | 100% |

### Language Recommendation

**TypeScript** — Full-stack with React/Vue frontend + Express/Fastify backend (or Next.js full-stack). The browser runtime requirement alone eliminates Rust, Julia, and C++. TypeScript's unmatched frontend ecosystem (D3.js, Plotly, Recharts) combined with type-safe full-stack development makes this the only practical choice.

### Architecture Sketch

```
Frontend (React + TypeScript):
[Upload Component] ──► axios POST /api/upload
                          │
[Chart Components] ◄──────┘
  ├── ConfusionMatrix (recharts heatmap)
  ├── ROCCurve (d3.js / plotly)
  ├── PrecisionRecall (plotly)
  ├── FeatureImportance (recharts bar)
  └── DataTable (tanstack-table with filtering)

Backend (Node.js + TypeScript):
[Express Server]
  ├── POST /api/upload ──► multer (file handling)
  │                         └── csv-parse / JSON.parse
  │                         └── compute statistics
  │                         └── cache in memory or Redis
  │
  ├── GET /api/stats/:id ──► return computed aggregations
  ├── GET /api/data/:id ──► return filtered/sorted data
  └── GET /api/export/:id ──► generate downloadable report
```

**Key packages:** `express`, `multer`, `csv-parse`, `d3`, `plotly.js`, `recharts`, `tanstack-table`, `zod`

---

## Example 3: Numerical Simulation for Differential Equations

### User Query

> "Write a numerical simulation for solving systems of ordinary differential equations. I need support for multiple methods (Euler, Runge-Kutta 4th order, Adams-Bashforth), adaptive step sizing, event detection (stop when condition met), and parallel ensemble simulation for parameter sweeps. Output should include solution trajectories and convergence diagnostics."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Scientific computing library for ODE numerical integration |
| **Core requirements** | Multiple ODE solvers, adaptive stepping, event detection, ensemble simulation, diagnostics |
| **Hidden requirements** | Type-generic solvers (work with different number types: Float32, Float64, Dual numbers for AD); composable solver interface; interpolation for dense output; stiff ODE support (implicit methods) |
| **Performance sensitivity** | High — numerical methods must be efficient; ensemble simulations should parallelize |
| **Deployment target** | Scientific computing environment (Jupyter notebooks, REPL, scripts) |
| **Ecosystem needs** | Linear algebra, automatic differentiation, plotting, statistical analysis |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Numerical computing ecosystem | 5 | 3 | **10** | 6 | 25% |
| Multiple dispatch for solvers | 4 | 2 | **10** | 4 | 15% |
| Development velocity (REPL, notebooks) | 5 | 5 | **10** | 4 | 15% |
| Parallel ensemble simulation | 7 | 4 | **9** | 8 | 10% |
| Automatic differentiation support | 6 | 3 | **10** | 5 | 10% |
| Plotting integration | 4 | 5 | **10** | 5 | 10% |
| Type stability for performance | 6 | 2 | **9** | 7 | 10% |
| **Weighted Score** | 5.3 | 3.3 | **9.8** | 5.7 | 100% |

### Language Recommendation

**Julia** — Designed for this exact use case. `DifferentialEquations.jl` ecosystem is the gold standard for ODE solving. Multiple dispatch enables elegant, performant solver composition. Native parallelism via `pmap`/`@distributed` for ensembles. `ForwardDiff.jl` for automatic differentiation. `Plots.jl` for visualization. REPL and Jupyter integration for interactive exploration.

### Architecture Sketch

```
[ODEProblem definition]
  ├── f(u, p, t) — right-hand side function
  ├── u0 — initial condition
  ├── tspan — time span
  └── p — parameters
         │
         ▼
[ solve() ]
  ├── Solver selection:
  │     ├── ODE.Euler() — simple, first-order
  │     ├── ODE.RK4() — classic 4th order
  │     ├── ODE.Tsit5() — adaptive, 5th order (default)
  │     └── ODE.Vern7() — higher accuracy
  │
  ├── Options:
  │     ├── adaptive stepping (tol, dt_max)
  │     ├── event detection (callback)
  │     └── dense output (interpolation)
  │
  └── solve(prob, solver, options) ──► Solution object
            │
            ▼
[EnsembleSimulation] ──► @distributed or pmap over parameter grid
            │
            ▼
[Analysis] ──► trajectories, convergence rates, parameter sensitivity
```

**Key packages:** `DifferentialEquations`, `ForwardDiff`, `Plots`, `BenchmarkTools`

---

## Example 4: Memory Allocator for Embedded System

### User Query

> "Implement a memory allocator for an embedded system with 64KB of RAM. Needs a fixed-size block allocator with 8, 16, 32, 64, and 128-byte pools. Must handle allocation failures gracefully (return null, no panic). Needs O(1) allocation and deallocation. No heap fragmentation. Must compile to bare-metal ARM Cortex-M4 with no_std."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Embedded systems memory allocator (no_std) |
| **Core requirements** | Fixed-size block pools, O(1) alloc/free, no fragmentation, `no_std`/`no_alloc`, ARM bare-metal |
| **Hidden requirements** | Thread-safety considerations (if RTOS used); deterministic behavior (no variance in timing); debug mode with usage statistics; potential defragmentation for variable-size future extension |
| **Performance sensitivity** | **Critical** — O(1) is a hard requirement; memory is extremely constrained |
| **Deployment target** | Bare-metal ARM Cortex-M4 (no OS, no standard library) |
| **Safety requirements** | No panics on allocation failure (embedded systems must keep running) |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| no_std / bare-metal support | **10** | N/A | N/A | 7 | 25% |
| Deterministic performance (no GC) | **10** | N/A | N/A | 9 | 20% |
| Memory safety without runtime | **10** | N/A | N/A | 4 | 20% |
| O(1) allocator implementation | **9** | N/A | N/A | 9 | 15% |
| Embedded ecosystem (HAL, PAC) | **9** | N/A | N/A | 6 | 10% |
| Compile-time size optimization | **8** | N/A | N/A | 7 | 10% |
| **Weighted Score** | **9.5** | N/A | N/A | 7.0 | 100% |

**Note:** TypeScript and Julia are N/A for bare-metal embedded. Between Rust and C++, Rust's `no_std` support with memory safety guarantees and embedded ecosystem (`cortex-m-rt`, `embedded-hal`) make it the clear choice.

### Language Recommendation

**Rust** — `no_std` + `no_alloc` is fully supported. `#![no_std]` crates ecosystem is mature for embedded. Ownership prevents use-after-free bugs that are catastrophic in embedded systems. `cortex-m-rt` crate handles startup. `embedded-hal` provides hardware abstraction. Compile-time memory layout control via `#[repr(C)]` and `core::alloc::Layout`.

### Architecture Sketch

```
[Memory Pool (statically allocated)]
  ├── Pool8:    256 blocks × 8 bytes   = 2,048 bytes
  ├── Pool16:   128 blocks × 16 bytes  = 2,048 bytes
  ├── Pool32:   64 blocks × 32 bytes   = 2,048 bytes
  ├── Pool64:   32 blocks × 64 bytes   = 2,048 bytes
  └── Pool128:  16 blocks × 128 bytes  = 2,048 bytes
         │
         ▼
[Allocator]
  ├── alloc(size) → find best-fit pool → pop from free list → return ptr
  │     └── O(1): size → pool index → free_list.pop()
  ├── dealloc(ptr) → determine pool from address → push to free list
  │     └── O(1): ptr → pool index → free_list.push()
  └── size classes: 8, 16, 32, 64, 128 (round up to nearest)

[Free List (intrusive linked list)]
  └── Each free block contains *next pointer
      └── No extra metadata overhead — pointer stored in free block itself
```

**Key features:** `#[no_std]`, `core::ptr`, `core::mem`, `spin` (for `Mutex` if threaded), `cortex-m` crate

---

## Example 5: REST API with Authentication

### User Query

> "Build a REST API with authentication. Users can register, log in with username/password, receive JWT tokens, and access protected endpoints. Need rate limiting, input validation, password hashing, and refresh tokens. Database for user storage. Should be deployable as a Docker container."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Web API with authentication and authorization |
| **Core requirements** | User registration, login, JWT tokens, protected routes, rate limiting, password hashing, refresh tokens, database |
| **Hidden requirements** | HTTPS/TLS termination; CORS configuration; input sanitization; audit logging; password reset flow; role-based access control (extensible); OpenAPI/Swagger docs |
| **Performance sensitivity** | Moderate — typical web API latency (< 200ms) |
| **Deployment target** | Docker container, cloud-native |
| **Ecosystem needs** | HTTP framework, auth/JWT library, ORM, database driver, validation, rate limiting |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Web framework maturity | 7 | **10** | 3 | 3 | 20% |
| Auth/JWT library ecosystem | 7 | **9** | 3 | 3 | 15% |
| Database ORM availability | 7 | **9** | 4 | 4 | 15% |
| Docker/deployment tooling | 7 | **9** | 4 | 5 | 10% |
| Input validation ecosystem | 8 | **9** | 4 | 4 | 10% |
| Type safety across stack | 8 | **8** | 5 | 5 | 10% |
| Development velocity | 6 | **9** | 5 | 4 | 10% |
| Runtime memory efficiency | **9** | 6 | 5 | 7 | 10% |
| **Weighted Score** | 7.2 | **8.9** | 3.8 | 4.0 | 100% |

### Language Recommendation

**TypeScript** — Mature ecosystem with `Express`/`Fastify`, `jsonwebtoken`, `bcrypt`, `Prisma` ORM, `zod` validation, `express-rate-limit`. Docker support is trivial (`node:20-alpine`). Development velocity is significantly higher than Rust for this CRUD+auth pattern. If the user explicitly requests maximum performance or memory efficiency, **Rust** (`axum` + `sqlx` + `argon2`) is a strong alternative with ~30% lower memory usage.

### Architecture Sketch

```
[Docker Container]
  ├── [Fastify HTTP Server]
  │     ├── POST /auth/register ──► validate ──► hash password ──► store user
  │     ├── POST /auth/login ──► verify password ──► issue JWT + refresh token
  │     ├── POST /auth/refresh ──► verify refresh token ──► issue new JWT
  │     ├── GET /api/profile ──► verify JWT ──► return user data
  │     └── GET /api/admin ──► verify JWT + role ──► return admin data
  │
  ├── [Rate Limiter] (Redis or in-memory)
  ├── [Zod Validation] (input schemas)
  ├── [Prisma ORM] ──► PostgreSQL
  └── [JWT Manager] (access + refresh tokens)
```

**Key packages:** `fastify`, `@fastify/jwt`, `@fastify/rate-limit`, `bcrypt` or `argon2`, `prisma`, `zod`, `dotenv`

---

## Example 6: Browser-Based AI Inference Demo

### User Query

> "Create a browser-based AI inference demo. Users can select a pre-trained image classification model (MobileNet, ResNet), upload an image, and see real-time classification results with confidence scores. Should run entirely client-side (no backend). Include a webcam mode for live classification."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Client-side web application running ML inference in the browser |
| **Core requirements** | Model loading, image preprocessing, inference execution, result display, webcam integration |
| **Hidden requirements** | Model quantization for speed (INT8/FP16); progressive model loading (show UI while model downloads); WebGL/WebGPU acceleration; graceful fallback if acceleration unavailable; offline capability (service worker); image preprocessing matching training pipeline (resize, normalize) |
| **Performance sensitivity** | High — inference must complete in < 500ms for interactive feel |
| **Deployment target** | Static web hosting (CDN, GitHub Pages, S3) |
| **Hard constraint** | Must run in browser — server execution is explicitly excluded |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Browser runtime (required) | N/A | **10** | N/A | N/A | 40% |
| ML inference in browser (ONNX/TF.js) | N/A | **10** | N/A | N/A | 25% |
| Webcam API access | N/A | **10** | N/A | N/A | 15% |
| Canvas/image processing | N/A | **9** | N/A | N/A | 10% |
| UI interactivity | N/A | **9** | N/A | N/A | 10% |
| **Weighted Score** | N/A | **9.8** | N/A | N/A | 100% |

**Note:** Browser runtime is a hard constraint. Only TypeScript/JavaScript runs natively in browsers. WebAssembly could theoretically enable Rust/C++, but for ML inference with webcam access, TypeScript with TensorFlow.js or ONNX Runtime Web is the production-proven path.

### Language Recommendation

**TypeScript** — `TensorFlow.js` for model execution with WebGL/WebGPU backends. ONNX Runtime Web as alternative for ONNX models. Browser APIs (`getUserMedia`, `Canvas`, `FileReader`) are natively available. React/Vanilla TypeScript for UI. Service worker for offline model caching.

### Architecture Sketch

```
[React TypeScript App]
  │
  ├── [Model Manager]
  │     ├── Download model from CDN (progressive)
  │     ├── Cache in IndexedDB
  │     └── Load into TensorFlow.js / ONNX Runtime
  │
  ├── [Image Input]
  │     ├── File upload → FileReader → Image → Canvas
  │     └── Webcam → getUserMedia → Video → Canvas
  │
  ├── [Preprocessor]
  │     ├── Resize to model input size (224×224)
  │     ├── Normalize (mean=[0.485, 0.456, 0.406], std=[...])
  │     └── Convert to Tensor
  │         │
  │         ▼
  ├── [Inference Engine]
  │     └── tf.model.predict(tensor) ──► logits ──► softmax
  │         │
  │         ▼
  ├── [Postprocessor]
  │     └── Top-K classes with confidence scores
  │         │
  │         ▼
  └── [Results Display]
        ├── Top prediction with confidence bar
        ├── Top-5 list with percentages
        └── Inference time (ms)
```

**Key packages:** `@tensorflow/tfjs`, `@tensorflow/tfjs-backend-webgl`, `onnxruntime-web`, `react`, `react-webcam`

---

## Example 7: Systems Monitoring Daemon

### User Query

> "Write a systems monitoring daemon that collects CPU, memory, disk, and network usage every 5 seconds. It should expose metrics via a Prometheus-compatible HTTP endpoint, log to syslog, handle configuration reload on SIGHUP, and run as a systemd service. Must be resource-efficient (minimal CPU and memory overhead)."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Long-running system daemon for metrics collection |
| **Core requirements** | System metrics collection, HTTP metrics endpoint, syslog logging, signal handling, systemd integration |
| **Hidden requirements** | Non-blocking metrics collection (don't hang if a subsystem is slow); configurable collection intervals; metric labels/tags for Prometheus; graceful shutdown on SIGTERM; optional eBPF for kernel-level metrics; memory-efficient circular buffers for recent data |
| **Performance sensitivity** | **Critical** — must use < 1% CPU on modern hardware; minimal memory footprint |
| **Deployment target** | Linux systemd systems (servers, VMs, containers) |
| **Runtime constraints** | Long-running process (days/weeks/months); no memory leaks; no crash on resource pressure |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Memory safety over long runtime | **10** | 5 | 3 | 5 | 25% |
| Systems programming (syscalls, signals) | **9** | 3 | 3 | 8 | 20% |
| Zero-cost async (tokio) | **9** | 6 | 3 | 6 | 15% |
| Binary size and startup time | **9** | 4 | 3 | 7 | 10% |
| Prometheus/client library ecosystem | **8** | 6 | 2 | 5 | 10% |
| systemd integration (sd-notify) | **8** | 4 | 2 | 6 | 10% |
| Cross-compilation for targets | **9** | 4 | 3 | 6 | 10% |
| **Weighted Score** | **9.0** | 4.5 | 2.8 | 6.3 | 100% |

### Language Recommendation

**Rust** — `tokio` for async runtime with minimal overhead. `sysinfo` crate for cross-platform system metrics. `prometheus` crate for metrics exposition. `tracing` for structured logging. Ownership guarantees zero memory leaks over months of uptime. `systemd` crate for sd-notify protocol. Static binary deployment with `cargo build --release --target x86_64-unknown-linux-musl`.

### Architecture Sketch

```
[main]
  ├── [Config Loader] ──► TOML config file
  │     └── SIGHUP handler ──► reload without restart
  │
  ├── [Metrics Collector] (tokio::time::interval every 5s)
  │     ├── CPU: /proc/stat parsing
  │     ├── Memory: /proc/meminfo parsing
  │     ├── Disk: statvfs syscall
  │     └── Network: /proc/net/dev parsing
  │
  ├── [Prometheus HTTP Server] (hyper/axum on port 9100)
  │     └── GET /metrics ──► render Prometheus text format
  │
  ├── [Logger] ──► tracing → syslog journald
  │
  └── [Signal Handlers]
        ├── SIGTERM ──► graceful shutdown
        ├── SIGHUP ──► config reload
        └── SIGINT ──► graceful shutdown
```

**Key crates:** `tokio`, `axum`, `prometheus`, `sysinfo`, `tracing`, `tracing-subscriber`, `signal-hook`, `serde`, `toml`

---

## Example 8: Matrix Computation Library for Scientific Computing

### User Query

> "Implement a matrix computation library for scientific computing. Need matrix multiplication, LU decomposition, QR decomposition, eigenvalue computation, and linear system solving. Should support Float32, Float64, and Complex numbers. Must be extensible to GPU backends. Include comprehensive tests comparing against reference implementations."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Numerical linear algebra library |
| **Core requirements** | BLAS-3 operations, LU/QR/eig decompositions, linear solves, generic over number types |
| **Hidden requirements** | LAPACK compatibility for advanced decompositions; GPU backend abstraction (CUDA/ROCm); sparse matrix support (future); distributed memory support (MPI, future); stability for ill-conditioned matrices; condition number estimation |
| **Performance sensitivity** | **Critical** — must match or approach BLAS/LAPACK performance; GPU acceleration expected for large matrices |
| **Deployment target** | HPC clusters, workstations, potentially cloud GPU instances |
| **Ecosystem needs** | BLAS/LAPACK bindings, GPU computing framework, automatic differentiation integration |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Weight |
|-----------|------|------------|-------|-----|--------|
| Linear algebra ecosystem | 6 | 3 | **10** | 7 | 25% |
| Multiple dispatch for types | 5 | 2 | **10** | 5 | 15% |
| BLAS/LAPACK integration | 6 | 2 | **10** | 8 | 15% |
| GPU computing (CUDA/ROCm) | 4 | 2 | **9** | 7 | 10% |
| Type-generic programming | 6 | 3 | **10** | 7 | 10% |
| HPC / cluster computing | 5 | 1 | **9** | 7 | 10% |
| Reference implementation testing | 6 | 3 | **9** | 6 | 10% |
| **Weighted Score** | 5.6 | 2.4 | **9.7** | 7.0 | 100% |

### Language Recommendation

**Julia** — Designed as a "numerical Python" with C-level performance. Built-in generic linear algebra via `LinearAlgebra` stdlib (which wraps OpenBLAS/MKL). Multiple dispatch enables `lu(A)`, `qr(A)`, `eigen(A)` to work seamlessly for `Float32`, `Float64`, `ComplexF32`, `ComplexF64`. `CUDA.jl` for GPU backends with minimal code changes. `GenericLinearAlgebra.jl` for pure-Julia fallback. Easy comparison against reference LAPACK implementations.

### Architecture Sketch

```
[Matrix Type]
  └── Matrix{T} where T <: Number
        ├── Float32, Float64, ComplexF32, ComplexF64
        └── Extensible to custom number types (Dual, BigFloat)

[BLAS Level 3] ──► gemm, syrk, trmm (via BLAS.jll)
  │
  ├── [LU Decomposition] ──► lu(A) → L, U, p
  │     └── pivoting for stability
  │
  ├── [QR Decomposition] ──► qr(A) → Q, R
  │     └── Householder reflections
  │
  ├── [Eigenvalue Decomposition] ──► eigen(A) → values, vectors
  │     └── Schur factorization (LAPACK.gees!)
  │
  ├── [Linear Solve] ──► A \ b
  │     └── uses LU for square, QR for least-squares
  │
  └── [GPU Backend] (CUDA.jl)
        └── cu(A) ──► CuMatrix ──► same operations on GPU

[Testing]
  ├── Unit tests: random matrices, known results
  ├── Property tests: A = LU, Q'Q = I, det(A) = prod(eigvals)
  ├── Reference comparison: vs. LAPACK reference, vs. NumPy/SciPy
  └── Performance benchmarks: vs. BLAS directly
```

**Key packages:** `LinearAlgebra` (stdlib), `GenericLinearAlgebra`, `CUDA`, `BenchmarkTools`, `Test`

---

## Example 9: Local AI-Glue Automation Service

### User Query

> "Build a local service that orchestrates several model-provider APIs and a vector store, exposes a small HTTP endpoint, persists events to SQLite, and is easy to extend with new modules week over week. Correctness and iteration speed matter more than raw throughput."

### Goal Analysis

| Aspect | Assessment |
|--------|-----------|
| **What it is** | Internal integration/automation service tying model APIs + a vector store together |
| **Core requirements** | Call multiple model-provider SDKs, query a vector store, expose a small HTTP API, persist events to SQLite, easy weekly extension |
| **Hidden requirements** | Retry/backoff on provider calls; structured logging; config via env; schema validation on inbound payloads; graceful degradation when a provider is down |
| **Performance sensitivity** | Low — correctness and iteration speed dominate; throughput is explicitly deprioritized |
| **Deployment target** | Local/server process, run as a long-lived service |
| **Ecosystem needs** | Model-provider SDKs, vector-store client, typed HTTP framework, lightweight persistence |

### Query Logic Analysis

| Criterion | Rust | TypeScript | Julia | C++ | Python | Weight |
|-----------|------|------------|-------|-----|--------|--------|
| AI/ML SDK ecosystem | 5 | 8 | 4 | 3 | **10** | 25% |
| Iteration speed / weekly extensibility | 6 | 8 | 6 | 3 | **10** | 20% |
| Backend / HTTP ecosystem | 7 | **10** | 4 | 4 | 8 | 15% |
| Vector-store / data libraries | 5 | 7 | 5 | 3 | **10** | 15% |
| Type safety where wanted | 9 | 9 | 5 | 6 | 7 | 10% |
| SQLite / persistence ergonomics | 7 | 8 | 6 | 5 | **9** | 10% |
| Raw throughput (deprioritized) | **10** | 6 | 7 | **10** | 4 | 5% |
| **Weighted Score** | 6.4 | 8.2 | 5.0 | 4.0 | **9.0** | 100% |

### Language Recommendation

**Python** — The model-provider SDKs and vector-store clients land in Python first, and the goal explicitly values iteration speed over throughput. `FastAPI` gives a typed, async HTTP layer; `pydantic` validates inbound payloads; `httpx` handles provider calls with retries; the stdlib `sqlite3` (or `sqlite-vec`) covers persistence. Type hints + `mypy` keep the dynamic codebase safe as it grows weekly. **TypeScript** is the runner-up and the better pick if this service must share a codebase with a TypeScript frontend.

### Architecture Sketch

```
[FastAPI app]
  ├── POST /ingest ──► pydantic-validated payload ──► event to SQLite
  ├── POST /query  ──► embed ──► vector-store search ──► model call ──► response
  └── GET  /health ──► provider + store status
         │
  ┌──────┴───────────────────────────────┐
  │ [Provider Clients] httpx + retry/backoff (model APIs)
  │ [Vector Store]     chromadb / sqlite-vec client
  │ [Persistence]      sqlite3 (events, append-only)
  │ [Modules]          drop-in handlers, registered at startup
  └──────────────────────────────────────┘
```

**Key packages:** `fastapi`, `uvicorn`, `httpx`, `pydantic`, `chromadb` (or `sqlite-vec`), `pytest`

---

## Decision Summary

| # | Request Type | Recommended Language | Key Differentiator |
|---|-------------|---------------------|-------------------|
| 1 | CLI log processor (large files) | **Rust** | Zero-allocation streaming, no GC |
| 2 | Web dashboard (visualization) | **TypeScript** | Browser runtime, chart ecosystem |
| 3 | ODE numerical simulation | **Julia** | DifferentialEquations.jl, multiple dispatch |
| 4 | Embedded memory allocator | **Rust** | no_std, bare-metal, ownership safety |
| 5 | REST API with auth | **TypeScript** | Web ecosystem velocity (Rust alternative) |
| 6 | Browser AI inference demo | **TypeScript** | Only browser-native option |
| 7 | Systems monitoring daemon | **Rust** | Long-running safety, tokio async |
| 8 | Matrix computation library | **Julia** | LinearAlgebra stdlib, GPU extensibility |
| 9 | Local AI-glue automation service | **Python** | AI SDK ecosystem + iteration speed |

### Language Selection Heuristics

| If the request involves... | Default to... | Consider alternative if... |
|---------------------------|---------------|---------------------------|
| AI/ML integration, automation, data pipelines, prototyping | **Python** | Hard real-time or throughput limits (→ Rust) |
| Systems programming, CLI tools, embedded | **Rust** | Legacy codebase is C++ |
| Web apps, APIs, browser, visualization | **TypeScript** | Extreme performance needed (→ Rust) |
| Numerical computing, scientific simulation, HPC | **Julia** | AI/ML glue or deployment ease (→ Python) |
| Game engines, real-time graphics, HPC | **C++** | Safety is paramount (→ Rust) |
| Bare-metal embedded, no_std | **Rust** | Existing C codebase (→ C/C++) |
