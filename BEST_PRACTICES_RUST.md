# Rust Best Practices for Claude Code Agent Swarm

> One-sentence rule: **Write code the compiler helps you keep correct, not code that fights the borrow checker.**

---

## Philosophy

- **Ownership is the model** — use it to express your program's data flow, not as an obstacle to work around.
- **Zero-cost abstractions** — prefer iterators and generics over hand-rolled loops when the intent is clearer; trust the optimizer.
- **Fearless concurrency** — use `Send` and `Sync` types; avoid shared mutable state unless you have a specific, well-reasoned concurrency primitive.

---

## Project Setup

### New Binary Project

```bash
cargo new project_name
cd project_name
```

### New Library Project

```bash
cargo new --lib project_name
```

### Workspace Structure

Use a workspace for multi-crate projects:

```
my-workspace/
├── Cargo.toml          # workspace manifest
├── crates/
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/lib.rs
│   ├── cli/
│   │   ├── Cargo.toml
│   │   └── src/main.rs
│   └── server/
│       ├── Cargo.toml
│       └── src/main.rs
```

**Root `Cargo.toml`:**

```toml
[workspace]
resolver = "2"
members = ["crates/*"]
```

---

## Required Commands

| Command | Purpose |
|---------|---------|
| `cargo new project_name` | Create a new binary crate |
| `cargo build` | Compile the project (dev profile) |
| `cargo test` | Run all tests |
| `cargo fmt` | Auto-format all code |
| `cargo clippy -- -D warnings` | Lint; treat warnings as errors |
| `cargo run` | Build and execute the binary |

**Always run the following before committing:**

```bash
cargo fmt && cargo clippy -- -D warnings && cargo test
```

---

## Optional Commands

| Command | Purpose |
|---------|---------|
| `cargo bench` | Run benchmarks (requires `bench = true` or `criterion`) |
| `cargo audit` | Scan dependencies for known security vulnerabilities |

---

## Code Style Rules

### Naming

| Item | Convention | Example |
|------|-----------|---------|
| Functions, variables, modules | `snake_case` | `compute_total`, `user_name` |
| Types, structs, enums, traits | `PascalCase` | `HttpRequest`, `ParserError` |
| Constants, statics | `SCREAMING_SNAKE_CASE` | `MAX_BUFFER_SIZE` |
| Generic parameters | `PascalCase`, single letter preferred | `T`, `K`, `V` |
| Lifetimes | `snake_case`, single letter | `'a`, `'ctx` |
| Macros | `snake_case!` | `println!`, `vec!` |

### Ownership and Borrowing Discipline

- **Borrow over clone** — pass `&T` or `&mut T` instead of cloning when the caller doesn't need ownership.
- **No unnecessary cloning** — every `.clone()` should have a comment or be obviously necessary.
- **Return ownership** — if a function produces a new value, return it; don't mutate an argument in place unless that is the documented contract.
- **Use `&str` over `String`** for function parameters that only need to read.
- **Use `&[T]` over `Vec<T>`** for function parameters that only need to read.

### Error Handling

- **Always use `Result<T, E>` for fallible operations.**
- **Libraries:** prefer `thiserror` for structured, typed errors:

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("missing field: {0}")]
    MissingField(String),
    #[error("invalid format")]
    InvalidFormat,
}
```

- **Applications:** prefer `anyhow` for ergonomic error propagation:

```rust
use anyhow::{Context, Result};

fn read_config(path: &str) -> Result<Config> {
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("failed to read config from {}", path))?;
    // ...
}
```

- **Avoid `unwrap()` and `expect()` outside tests.** If you must use them, add a `// SAFETY:` or `// REASON:` comment explaining why the condition is guaranteed.

### Serialization

- Use `serde` when you need to serialize/deserialize:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    name: String,
    port: u16,
}
```

### Async

- **Use `tokio` only when async is justified** — network servers, concurrent I/O, or when integrating with an async ecosystem.
- **Do not use async for CPU-bound work.** Use `rayon` or plain threads for parallelism.
- If the project has no async dependency, do not introduce `tokio` just for "future-proofing."

### Traits and Generics

- **Use traits only when they simplify the design** — don't create a trait for a single implementation.
- **Avoid premature lifetimes and generics** — start with concrete types; introduce generics when you have a real duplication problem.
- **Prefer simple structs and enums** over trait-heavy architectures.

### Benchmarking

- Include benchmarks only when performance matters. Use `criterion` for statistically rigorous benchmarks:

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib 20", |b| b.iter(|| fibonacci(black_box(20))));
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

---

## Security Rules

1. **No `unsafe` code unless justified and documented.** Every `unsafe` block needs a `// SAFETY:` comment explaining the invariants upheld.
2. **Validate all external input** — files, network data, environment variables. Never trust input shape or size.
3. **Run `cargo audit` regularly** to check for known vulnerabilities in dependencies.
4. **Minimize dependency count** — each crate is a trust boundary.
5. **Do not hardcode secrets.** Use environment variables or a secret manager.

---

## Testing Rules

- **Unit tests** — inline in the source file, inside `#[cfg(test)]` modules:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
}
```

- **Integration tests** — in the `tests/` directory at project root, one file per feature area.
- **Doc tests** — write examples in `///` doc comments for all public APIs:

```rust
/// Adds two numbers.
///
/// # Examples
///
/// ```
/// use mycrate::add;
/// assert_eq!(add(2, 3), 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

---

## Project Structure Template

```
my-project/
├── Cargo.toml
├── README.md
├── src/
│   ├── main.rs          # or lib.rs
│   ├── config.rs        # configuration parsing
│   ├── error.rs         # error types (thiserror)
│   ├── models.rs        # core data structures
│   └── utils.rs         # shared helpers
├── tests/
│   ├── integration_test.rs
│   └── fixtures/
│       └── sample.json
├── benches/
│   └── my_bench.rs
└── .github/
    └── workflows/
        └── ci.yml       # fmt, clippy, test, audit
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `String` parameter instead of `&str` | Forces allocation for string literals | Use `&str` |
| `.clone()` in a loop without checking | Unnecessary allocations | Restructure to borrow |
| `unwrap()` in production code | Panics crash the program | Use `?` or `match` |
| `Rc<RefCell<T>>` for everything | Overcomplicated, runtime overhead | Use ownership or `Arc<Mutex<T>>` if truly shared |
| Async for CPU work | Wrong tool for parallelism | Use `rayon` or threads |
| Giant `main()` function | Untestable, unclear | Extract functions, use a library crate |
| Ignoring `Result` with `let _ =` | Silent failures | Handle or propagate with `?` |
| Global mutable state | Hard to test, race conditions | Pass state explicitly |
| Overuse of lifetimes | Complexity without benefit | Clone if simpler; use `Arc` or owned types |
| Every struct gets a trait | Indirection without value | Use concrete types until duplication is real |

---

## Example: Minimal Correct Rust Project

```
greet/
├── Cargo.toml
└── src/
    ├── main.rs
    └── lib.rs
```

**`Cargo.toml`**

```toml
[package]
name = "greet"
version = "0.1.0"
edition = "2021"

[dependencies]
anyhow = "1"
```

**`src/lib.rs`**

```rust
use anyhow::{bail, Result};

/// Returns a greeting string for the given name.
///
/// # Examples
///
/// ```
/// use greet::greet;
/// assert_eq!(greet("Ada").unwrap(), "Hello, Ada!");
/// ```
///
/// # Errors
///
/// Returns an error if the name is empty.
pub fn greet(name: &str) -> Result<String> {
    if name.is_empty() {
        bail!("name must not be empty");
    }
    Ok(format!("Hello, {}!", name))
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_greet_ok() {
        assert_eq!(greet("Ada").unwrap(), "Hello, Ada!");
    }

    #[test]
    fn test_greet_empty() {
        assert!(greet("").is_err());
    }
}
```

**`src/main.rs`**

```rust
use anyhow::Result;
use greet::greet;

fn main() -> Result<()> {
    let name = std::env::args()
        .nth(1)
        .unwrap_or_else(|| "World".to_string());
    println!("{}", greet(&name)?);
    Ok(())
}
```

**Run the example:**

```bash
cargo fmt
cargo clippy -- -D warnings
cargo test
cargo run -- "Ada"
```

---

## Quick Reference

```bash
# Full pre-commit check
cargo fmt && cargo clippy -- -D warnings && cargo test && cargo audit

# Run with backtrace on panic
RUST_BACKTRACE=1 cargo run

# Run a single test
cargo test test_greet_ok -- --nocapture

# Check without building
cargo check

# Build release
cargo build --release
```
