# Rust-Specific Claude Code Prompt

Append this prompt after `PROMPT_UNIVERSAL.md` for all Rust projects. These rules supplement -- not replace -- the universal workflow.

---

## Rust Build Commands

Use these commands throughout the workflow:

| Command | Purpose | When to run |
|---------|---------|-------------|
| `cargo new project_name` | Create new binary project | Project initialization |
| `cargo new --lib project_name` | Create new library project | Library initialization |
| `cargo build` | Compile project | After code changes |
| `cargo build --release` | Optimized release build | Before benchmarking |
| `cargo test` | Run all tests | After implementation, must pass |
| `cargo test -- --nocapture` | Run tests with stdout | Debugging test output |
| `cargo fmt` | Auto-format code | Before every commit |
| `cargo clippy -- -D warnings` | Lint (warnings as errors) | Before every commit, must pass |
| `cargo run` | Build and execute | Manual verification |
| `cargo run --release` | Run optimized build | Performance verification |
| `cargo bench` | Run benchmarks | Step 13 (Optimization Pass) |
| `cargo doc --open` | Generate documentation | Documentation review |
| `cargo audit` | Check dependencies for vulnerabilities | Security review |
| `cargo check` | Fast syntax/type check | Quick feedback loop |

---

## Rust Code Rules

### Ownership and Borrowing
- Write idiomatic Rust using ownership and borrowing correctly.
- No unnecessary `.clone()` calls. If you clone, justify why borrowing won't work.
- Use references (`&T`, `&mut T`) instead of moving values when the caller needs the value afterward.
- Prefer `&str` over `String` for function arguments. Prefer `&[T]` over `Vec<T>`.
- Use lifetimes only when necessary. Prefer owning types (`String`, `Vec`, `Box`) at API boundaries.

### Error Handling
- Use `Result<T, E>` for all fallible operations. No silently ignoring errors.
- `unwrap()` and `expect()` are allowed **only in tests** or **with explicit justification** (e.g., invariant guaranteed by construction).
- Prefer `thiserror` for application errors with structured variants.
- Prefer `anyhow` for CLI/tool binaries where error context matters more than matching.
- Include error source chains: `#[from] std::io::Error` or `.context("...")`.
- Error messages must be actionable: tell the user what went wrong and what they can do.

### Types and Traits
- Use `serde` for serialization/deserialization when data crosses boundaries (files, network, CLI args).
- Derive common traits: `Debug`, `Clone`, `PartialEq` where appropriate. Derive `Default` for configuration types.
- Use `std::collections` appropriately: `HashMap` for key-value, `BTreeMap` for ordered, `Vec` for sequences.
- Prefer enums with data over stringly-typed values.
- Use `Option<T>` and `Result<T, E>` at API boundaries instead of sentinel values.

### Async
- Use `tokio` only when async is justified: I/O-bound workloads, multiple concurrent connections, or async ecosystem requirements.
- Do not use async for CPU-bound work. Use `rayon` for parallel computation instead.
- If using async, prefer `tokio::spawn` for true concurrency. Use `join!` for independent futures.
- Mark blocking operations in async contexts with `spawn_blocking`.

### Performance
- Avoid premature optimization. Profile first, optimize second.
- Use `cargo bench` (via `criterion` or `divan`) for benchmarks. Include before/after numbers.
- Use zero-copy parsing (`&str` slices) for text processing when possible.
- Use `Vec::with_capacity` when the size is known.
- Consider `Cow<'_, str>` for functions that may or may not need to allocate.

### Generics and Lifetimes
- Avoid premature generics. Start concrete, abstract when duplication appears.
- Avoid lifetime parameters in public APIs unless necessary. Use owning types at boundaries.
- Do not use `PhantomData` unless you are implementing unsafe code or advanced patterns.

---

## Rust Security

- **No `unsafe` unless justified.** If `unsafe` is required, document: (1) why safe Rust cannot do it, (2) the safety invariant, (3) the proof that the invariant is maintained, (4) who is responsible for upholding it.
- Run `cargo audit` before final delivery. Address or document all reported vulnerabilities.
- No secrets in source code. Use environment variables or secret management.
- Validate all external input at the crate boundary (files, network, user input).
- Use constant-time comparison for cryptographic operations (if applicable).

---

## Rust Testing

### Test Organization
```
src/
  lib.rs          -- Library root, re-exports
  module_a.rs     -- Module with inline #[cfg(test)] unit tests
  module_b.rs     -- Module with inline #[cfg(test)] unit tests
tests/
  integration.rs  -- Integration tests (test public API)
benches/
  bench.rs        -- Criterion benchmarks (if performance matters)
examples/
  basic.rs        -- Runnable usage example
Cargo.toml
```

### Unit Tests
- Write unit tests inline using `#[cfg(test)] mod tests` at the bottom of each source file.
- Test every public function with at least one success case and one error case.
- Name tests descriptively: `fn rejects_negative_input()` not `fn test1()`.
- Use `assert_eq!`, `assert_ne!`, `assert!(...)` with descriptive messages.

### Integration Tests
- Place in `tests/` directory. Test the public API as an external consumer would.
- Use `super::*` or explicit imports. Do not test private functions.
- Test complete workflows end-to-end.

### Doc Tests
- Include `/// # Examples` sections in public API documentation.
- Doc tests must compile and pass (`cargo test --doc`).
- Show realistic usage, not toy examples.

### Test Data
- Use explicit test fixtures. No magic numbers without explanation.
- Create helper functions for common setup. Use `rstest` or parameterized tests for matrix testing.

---

## Rust Project Structure Template

Use this structure unless the project demands otherwise:

```
project_name/
  Cargo.toml          -- Dependencies, metadata, [workspace] if multi-crate
  rust-toolchain.toml -- Pin toolchain version (optional but recommended)
  .rustfmt.toml       -- Formatting config
  src/
    main.rs / lib.rs  -- Entry point, module declarations
    config.rs         -- Configuration parsing and validation
    error.rs          -- Error types (using thiserror)
    cli.rs            -- CLI argument parsing (using clap)
    core_logic.rs     -- Pure business logic (no I/O)
    io.rs             -- File/network/database I/O
  tests/
    integration.rs    -- End-to-end tests
  benches/
    benchmark.rs      -- Performance benchmarks
  examples/
    basic.rs          -- Usage example
  .gitignore
  README.md           -- Setup, build, test, usage instructions
  LICENSE
```

### Cargo.toml Requirements

```toml
[package]
name = "project_name"
version = "0.1.0"
edition = "2021"
license = "MIT OR Apache-2.0"
repository = "https://github.com/..."
rust-version = "1.78"  # MSRV

[dependencies]
# Every dependency must be justified in the Architecture Plan
serde = { version = "1", features = ["derive"] }
thiserror = "1"

[dev-dependencies]
criterion = "0.5"     # For benchmarks
tempfile = "3"        # For test file fixtures

[[bench]]
name = "benchmark"
harness = false

[profile.release]
lto = true            # Enable link-time optimization for releases
```

---

## Rust Review Checklist

Before marking a Rust project complete, verify every item:

### Code Quality
- [ ] `cargo fmt` produces no changes
- [ ] `cargo clippy -- -D warnings` passes cleanly
- [ ] No compiler warnings (even without `-D warnings`)
- [ ] All `unwrap()`/`expect()` calls are justified or in tests
- [ ] No unnecessary `.clone()` calls
- [ ] Idiomatic use of ownership and borrowing
- [ ] Error types are descriptive and actionable
- [ ] Public API has doc comments with examples
- [ ] Private functions have doc comments if logic is non-trivial

### Testing
- [ ] `cargo test` passes (all unit, integration, doc tests)
- [ ] Every public function has at least one test
- [ ] Error paths are tested
- [ ] Edge cases are tested (empty input, max values, boundary conditions)
- [ ] Integration tests cover end-to-end workflows

### Performance
- [ ] Benchmarks exist if performance matters
- [ ] No premature optimization without measurement
- [ ] Performance-critical sections are documented

### Security
- [ ] No `unsafe` blocks, or each is fully documented with safety proof
- [ ] `cargo audit` passes or all findings are documented
- [ ] No secrets in source code
- [ ] All external input is validated

### Documentation
- [ ] README includes: what it does, how to build, how to test, how to run
- [ ] Examples compile and run
- [ ] `cargo doc` generates clean documentation with no broken links

---

## Rust-Specific Decision Guide

| Decision | Prefer | Avoid |
|----------|--------|-------|
| Error handling in library | `thiserror` + custom enum | `anyhow` (opaque for libraries) |
| Error handling in CLI/binary | `anyhow` | Raw `Result<(), Box<dyn Error>>` |
| CLI parsing | `clap` (derive feature) | Manual argv parsing |
| Serialization | `serde` + `serde_json`/`toml` | Manual parsing |
| Async runtime | `tokio` (justified only) | Unnecessary async |
| Parallel CPU work | `rayon` | Async for CPU-bound |
| HTTP client | `reqwest` (sync or async) | Manual HTTP |
| Testing framework | Built-in `cargo test` | External test runners |
| Benchmarking | `criterion` or `divan` | Ad-hoc timing |
| Temporary files in tests | `tempfile` crate | Manual cleanup |
| Configuration | `config` + `serde` | Manual file parsing |
| Logging | `tracing` (structured) | `println!` debugging |
