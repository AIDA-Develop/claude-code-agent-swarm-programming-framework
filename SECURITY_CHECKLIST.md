# Security Checklist

> **Purpose:** This checklist is used by the **Security Review Agent** to verify that all code produced by the swarm does not introduce security vulnerabilities, leak sensitive information, or create unnecessary attack surfaces.
>
> **Workflow:** Review code against all sections → Mark each item → Record any findings → Compute risk level → Require sign-off before delivery.

---

## Risk Classification

| Level | Definition | Action |
|-------|-----------|--------|
| **CRITICAL** | Exploitable vulnerability in production code | **BLOCK DELIVERY** - must fix before release |
| **HIGH** | Significant security concern, potential vulnerability | Must fix before release |
| **MEDIUM** | Security improvement recommended | Should fix in follow-up |
| **LOW** | Minor security hygiene issue | Document, fix when convenient |
| **INFO** | Awareness item, no immediate action | Note for future reference |

---

## 1. Input Validation

> **Rule:** All untrusted input must be validated before processing. Trust nothing from outside the process boundary.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 1.1 | All user-provided input is validated (length, type, range, format) | [ ] | CRITICAL |
| 1.2 | File paths are canonicalized and checked for traversal (`../`) | [ ] | HIGH |
| 1.3 | URL/URI parameters are validated before use | [ ] | HIGH |
| 1.4 | Environment variables are validated before use (not blindly trusted) | [ ] | MEDIUM |
| 1.5 | Command-line arguments are validated (not passed unchecked to shell) | [ ] | HIGH |
| 1.6 | Uploaded files are validated (type, size, content—not just extension) | [ ] | HIGH |
| 1.7 | Deserialization of untrusted data uses safe methods | [ ] | HIGH |
| 1.8 | Regular expressions are safe (no ReDoS—catastrophic backtracking) | [ ] | MEDIUM |
| 1.9 | Numeric inputs are bounded (no integer overflow/underflow) | [ ] | HIGH |
| 1.10 | String inputs have maximum length enforced | [ ] | MEDIUM |

### Validation Pattern Template

```
WHEN receiving [input type]:
  1. Check type/structure
  2. Check length/range
  3. Check allowed characters/format (whitelist > blacklist)
  4. Reject or sanitize BEFORE processing
  5. Log validation failure at appropriate level
```

---

## 2. Dependencies

> **Rule:** Minimize attack surface. Every dependency is a trust anchor and a potential vulnerability.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 2.1 | All dependencies are necessary (no unused dependencies) | [ ] | MEDIUM |
| 2.2 | Dependencies have no known CVEs (audit tool run) | [ ] | CRITICAL |
| 2.3 | Dependencies are from trusted sources (official registries) | [ ] | HIGH |
| 2.4 | Dependency versions are pinned (lockfile committed) | [ ] | MEDIUM |
| 2.5 | No dependencies on unmaintained or deprecated packages | [ ] | MEDIUM |
| 2.6 | Transitive dependency tree has been reviewed (`cargo tree`, `npm ls`) | [ ] | LOW |
| 2.7 | No dependencies using Git branches or unversioned sources in production | [ ] | MEDIUM |
| 2.8 | Supply chain: dependencies are the actual intended packages (not typosquats) | [ ] | HIGH |

### Dependency Audit Commands

| Language | Audit Command |
|----------|--------------|
| **Rust** | `cargo audit` (via `cargo-audit` crate) |
| **TypeScript** | `npm audit` or `yarn audit` |
| **Julia** | `Pkg.status()` + manual registry verification |
| **C++** | Manual review; Conan/vcpkg checksum verification |

---

## 3. Secrets Management

> **Rule:** No secrets in code. Ever. Not even "temporary" ones.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 3.1 | No hardcoded passwords, API keys, tokens, or credentials | [ ] | CRITICAL |
| 3.2 | No secrets in comments or documentation | [ ] | CRITICAL |
| 3.3 | No secrets in test files (use test fixtures or environment vars) | [ ] | CRITICAL |
| 3.4 | Configuration loaded from environment variables or secure vaults | [ ] | HIGH |
| 3.5 | `.env` files are in `.gitignore` (never committed) | [ ] | CRITICAL |
| 3.6 | Log files do not contain secrets or tokens | [ ] | HIGH |
| 3.7 | Error messages do not expose secrets or internal paths | [ ] | HIGH |
| 3.8 | CI/CD pipelines use encrypted secrets (not plaintext env vars) | [ ] | MEDIUM |

### Secrets Detection Checklist

Run these before every commit:

```bash
# Check for high-entropy strings that look like keys
grep -rnE '(api[_-]?key|password|secret|token)\s*[=:]\s*["\'][^"\']{16,}["\']' src/

# Check for common key patterns
grep -rnE '(AKIA[0-9A-Z]{16}|ghp_[a-zA-Z0-9]{36}|sk-[a-zA-Z0-9]{48})' src/

# Check .gitignore has .env
grep -q '^\.env' .gitignore || echo "WARNING: .env not in .gitignore"
```

---

## 4. Error Information Leakage

> **Rule:** Error messages help users, not attackers. Internal details stay internal.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 4.1 | Error messages to users are generic (no stack traces, SQL, paths) | [ ] | HIGH |
| 4.2 | Stack traces are logged internally but not exposed to users | [ ] | HIGH |
| 4.3 | Database error messages are not forwarded to users | [ ] | CRITICAL |
| 4.4 | Internal file paths are not exposed in error messages | [ ] | MEDIUM |
| 4.5 | Server/framework version numbers are not exposed in headers/responses | [ ] | LOW |
| 4.6 | Debug mode is disabled in production | [ ] | HIGH |
| 4.7 | Detailed error info is logged server-side for debugging | [ ] | INFO |

### Error Message Classification

| Audience | Content Example |
|----------|----------------|
| **End user** | "Invalid username or password" |
| **Developer** | "Authentication failed: bcrypt hash mismatch for user_id=123" |
| **Attacker** | "SQL Error 1064 near 'admin'--'" (NEVER show this) |

---

## 5. Unsafe Code

> **Rule:** Unsafe code must be justified, documented, and minimized. Safety invariants must be explicit.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 5.1 | Unsafe blocks are the minimum size possible | [ ] | HIGH |
| 5.2 | Every unsafe block has a `// SAFETY:` comment explaining invariants | [ ] | CRITICAL |
| 5.3 | Unsafe code has been reviewed by a second agent | [ ] | HIGH |
| 5.4 | Safe alternatives were considered before using unsafe | [ ] | MEDIUM |
| 5.5 | Unsafe code is isolated behind a safe API | [ ] | HIGH |
| 5.6 | Miri (`cargo miri test`) passes for unsafe code | [ ] | HIGH |
| 5.7 | No undefined behavior (verified with Miri, ASan, UBSan) | [ ] | CRITICAL |
| 5.8 | Raw pointers are used only when references cannot work | [ ] | MEDIUM |

### `// SAFETY:` Comment Template

```rust
// SAFETY: We hold a unique reference to `inner`, and `len` was just verified
// to be <= capacity. The pointer is valid for writes of `len` bytes because
// we allocated `capacity` bytes on creation and haven't deallocated.
unsafe { ptr::write(ptr.add(index), value) };
```

---

## 6. Concurrency Safety

> **Rule:** Concurrent code must be free of data races, deadlocks, and race conditions.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 6.1 | No data races (verified via type system, TSan, or Miri) | [ ] | CRITICAL |
| 6.2 | No deadlock potential (lock ordering is consistent) | [ ] | HIGH |
| 6.3 | No race conditions in shared state access | [ ] | HIGH |
| 6.4 | Lock scopes are minimal (no holding locks across I/O) | [ ] | MEDIUM |
| 6.5 | Async code does not block the executor | [ ] | HIGH |
| 6.6 | Thread pools have bounded size (no unbounded spawning) | [ ] | MEDIUM |
| 6.7 | `Send`/`Sync` traits are implemented only when safe (Rust) | [ ] | CRITICAL |
| 6.8 | Atomics use correct memory ordering | [ ] | HIGH |

---

## 7. Injection Prevention

> **Rule:** Never compose commands or queries with string concatenation from untrusted input.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 7.1 | No string concatenation into SQL queries (use parameterized queries) | [ ] | CRITICAL |
| 7.2 | No shell command construction from user input (use execve-style APIs) | [ ] | CRITICAL |
| 7.3 | No path traversal via user input (canonicalize + whitelist) | [ ] | HIGH |
| 7.4 | No HTML/JS injection in rendered output (escape all user content) | [ ] | CRITICAL |
| 7.5 | No XPath/LDAP injection (use parameterized APIs) | [ ] | HIGH |
| 7.6 | No XML external entity (XXE) attacks (disable external entities) | [ ] | HIGH |
| 7.7 | Template engines auto-escape by default | [ ] | HIGH |
| 7.8 | No eval/Function/constructor on dynamic strings (TypeScript) | [ ] | CRITICAL |

---

## 8. Cryptography

> **Rule:** Do not roll your own crypto. Use established, well-reviewed libraries.

| # | Check Item | Status | Risk |
|---|-----------|--------|------|
| 8.1 | No custom cryptographic algorithms or protocols | [ ] | CRITICAL |
| 8.2 | No custom hash functions | [ ] | CRITICAL |
| 8.3 | No custom random number generators for security purposes | [ ] | CRITICAL |
| 8.4 | Passwords are hashed with bcrypt, Argon2, or scrypt (NOT SHA-256) | [ ] | CRITICAL |
| 8.5 | HTTPS/TLS is used for all network communication | [ ] | HIGH |
| 8.6 | Cryptographic keys are loaded from secure storage (not hardcoded) | [ ] | CRITICAL |
| 8.7 | Nonce/IV values are unique per encryption operation | [ ] | HIGH |
| 8.8 | Cryptographic libraries are current (no deprecated algorithms: MD5, SHA-1, DES) | [ ] | HIGH |

### Approved Cryptographic Libraries

| Language | Approved Libraries |
|----------|-------------------|
| **Rust** | `ring`, `rustls`, `argon2`, `sha2` (from RustCrypto) |
| **TypeScript** | Node.js `crypto` module, `libsodium-wrappers` |
| **Julia** | `MbedTLS.jl`, `SHA.jl` (stdlib) |
| **C++** | OpenSSL (careful), `libsodium`, Botan |

---

## 9. Per-Language Security Specifics

### 9.1 Rust Security Checklist

| # | Check Item | Status |
|---|-----------|--------|
| 9.1.1 | `cargo audit` passes with no vulnerabilities | [ ] |
| 9.1.2 | `unsafe` blocks have `// SAFETY:` comments | [ ] |
| 9.1.3 | Miri passes on unsafe code (`cargo miri test`) | [ ] |
| 9.1.4 | Clippy security lints are enabled (`#![deny(unsafe_op_in_unsafe_fn)]`) | [ ] |
| 9.1.5 | No `unwrap()` / `expect()` on user-controlled data | [ ] |
| 9.1.6 | Panic messages don't leak sensitive data | [ ] |
| 9.1.7 | `std::process::Command` arguments passed as list, not string | [ ] |
| 9.1.8 | File permissions are set explicitly (not relying on umask) | [ ] |

### 9.2 TypeScript Security Checklist

| # | Check Item | Status |
|---|-----------|--------|
| 9.2.1 | `npm audit` passes with no high/critical vulnerabilities | [ ] |
| 9.2.2 | No `eval()`, `Function()`, or `setTimeout(string)` | [ ] |
| 9.2.3 | No prototype pollution (Object.freeze on inputs; no `__proto__` assignment) | [ ] |
| 9.2.4 | XSS prevention: all user output is escaped in HTML contexts | [ ] |
| 9.2.5 | CSRF tokens used for state-changing operations | [ ] |
| 9.2.6 | Content Security Policy headers are set | [ ] |
| 9.2.7 | `helmet` or equivalent security middleware is used (Express/Fastify) | [ ] |
| 9.2.8 | `strict` mode is enabled in `tsconfig.json` | [ ] |
| 9.2.9 | No `child_process.exec` with user input (use `execFile` with array args) | [ ] |
| 9.2.10 | Authentication cookies use `httpOnly`, `secure`, `sameSite` | [ ] |

### 9.3 Julia Security Checklist

| # | Check Item | Status |
|---|-----------|--------|
| 9.3.1 | Package sources verified (official General registry) | [ ] |
| 9.3.2 | Input sanitized before `run()` or file operations | [ ] |
| 9.3.3 | `@eval` and `Meta.parse` not used on user input | [ ] |
| 9.3.4 | No arbitrary code execution via deserialized data | [ ] |
| 9.3.5 | Temporary files use `mktemp()` (not predictable names) | [ ] |
| 9.3.6 | Downloads use `Downloads.jl` with verified URLs | [ ] |
| 9.3.7 | No global mutable state accessible to untrusted code | [ ] |

### 9.4 C++ Security Checklist

| # | Check Item | Status |
|---|-----------|--------|
| 9.4.1 | ASan (AddressSanitizer) passes: `-fsanitize=address` | [ ] |
| 9.4.2 | UBSan passes: `-fsanitize=undefined` | [ ] |
| 9.4.3 | No buffer overflows (bounds checking or safe containers) | [ ] |
| 9.4.4 | No use-after-free (smart pointers, RAII) | [ ] |
| 9.4.5 | No format string vulnerabilities (`printf` with user data → `fmt` library) | [ ] |
| 9.4.6 | No integer overflow in security-sensitive calculations (`-ftrapv` or checked) | [ ] |
| 9.4.7 | Stack protector enabled (`-fstack-protector-strong`) | [ ] |
| 9.4.8 | FORTIFY_SOURCE enabled (`-D_FORTIFY_SOURCE=2`) | [ ] |
| 9.4.9 | No raw `new`/`delete` (use `std::make_unique`/`make_shared`) | [ ] |
| 9.4.10 | No `strcpy`, `strcat`, `sprintf` (use `strncpy`, `snprintf`, or `std::string`) | [ ] |

---

## 10. Security Review Sign-Off Template

```markdown
# Security Review Sign-Off

## Review Information

**Review Date:** ___________
**Reviewer Agent:** ___________
**Code Language(s):** ___________
**Files Reviewed:** ___________

## Findings Summary

| Severity | Count | Status |
|----------|-------|--------|
| CRITICAL | ___ | ___ resolved, ___ open |
| HIGH | ___ | ___ resolved, ___ open |
| MEDIUM | ___ | ___ resolved, ___ open |
| LOW | ___ | ___ resolved, ___ open |

## Critical / High Findings

### 1. [TITLE]
- **Severity:** ___
- **Location:** `file.rs:42`
- **Description:** ___________
- **Remediation:** ___________
- **Status:** [ ] Resolved  [ ] Open  [ ] Accepted Risk

### 2. [TITLE]
- **Severity:** ___
- **Location:** `file.rs:67`
- **Description:** ___________
- **Remediation:** ___________
- **Status:** [ ] Resolved  [ ] Open  [ ] Accepted Risk

## Checklist Verification

| Section | Passed |
|---------|--------|
| Input Validation | [ ] |
| Dependencies | [ ] |
| Secrets Management | [ ] |
| Error Information Leakage | [ ] |
| Unsafe Code | [ ] |
| Concurrency Safety | [ ] |
| Injection Prevention | [ ] |
| Cryptography | [ ] |
| Per-Language Checks | [ ] |

## Sign-Off

**Overall Risk Level:** [ ] LOW  [ ] MEDIUM  [ ] HIGH  [ ] CRITICAL

**Approved for delivery:** [ ] Yes  [ ] No  [ ] Conditionally (see below)

**Conditions (if conditional approval):**
___________________________________________________________
___________________________________________________________

**Reviewer Signature:** ___________
**Date:** ___________
```

---

## Quick Reference: Common Security Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| `eval(user_input)` | Arbitrary code execution | Parse/sanitize input; use safe APIs |
| `SELECT * FROM users WHERE id = '` + id | SQL injection | Parameterized queries |
| `system("rm -rf " + user_path)` | Command injection | Use `std::process::Command` with array args |
| `fs.readFileSync(user_path)` | Path traversal | Canonicalize + whitelist allowed paths |
| `return { error: err.stack }` | Info leak | Return generic message; log details server-side |
| `const hash = sha256(password)` | Weak password hashing | Use bcrypt, Argon2, or scrypt |
| `Math.random()` for crypto | Predictable randomness | Use `crypto.randomBytes()` or `getrandom` |
| `let token = "hardcoded_secret_123"` | Secret in code | Load from env var or secrets manager |
| `innerHTML = userContent` | XSS | Use `textContent` or sanitize with DOMPurify |
| `unsafe { *ptr }` without bounds check | Use-after-free / segfault | Check bounds; use safe abstractions |
