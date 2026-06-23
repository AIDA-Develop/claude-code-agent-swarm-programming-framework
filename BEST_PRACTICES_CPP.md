# C++ Best Practices for Claude Code Agent Swarm

> One-sentence rule: **Prefer value semantics, RAII, and the standard library. Never own memory with a raw pointer.**

---

## Philosophy

- **Modern C++** — use the language as of C++20/23. Avoid pre-2011 patterns.
- **RAII (Resource Acquisition Is Initialization)** — every resource (memory, file handles, locks) is owned by an object whose destructor releases it.
- **Zero-overhead principle** — you don't pay for what you don't use. But don't pretend C++ is C with classes.
- **Safety-aware** — modern C++ can be safe. Use the tools: smart pointers, `std::optional`, `std::expected`, bounds-checked access.

---

## Project Setup

### Directory Layout

```
my-project/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   ├── config.hpp
│   └── config.cpp
├── include/
│   └── myproject/
│       └── core.hpp
├── tests/
│   └── test_core.cpp
└── .clang-format
```

### CMakeLists.txt Template

```cmake
cmake_minimum_required(VERSION 3.20)
project(my-project VERSION 0.1.0 LANGUAGES CXX)

# C++23 by default. Override only if required.
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Warnings as errors for clean code
if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()

# Library
add_library(mylib
    src/config.cpp
)
target_include_directories(mylib PUBLIC include)

# Executable
add_executable(myapp src/main.cpp)
target_link_libraries(myapp PRIVATE mylib)

# Tests — enabled when built as top-level project
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    enable_testing()
    add_subdirectory(tests)
endif()
```

### tests/CMakeLists.txt

```cmake
# Using doctest as a single-header test framework
include(FetchContent)
FetchContent_Declare(
    doctest
    GIT_REPOSITORY https://github.com/doctest/doctest.git
    GIT_TAG v2.4.11
)
FetchContent_MakeAvailable(doctest)

add_executable(test_core test_core.cpp)
target_link_libraries(test_core PRIVATE mylib doctest::doctest)
add_test(NAME core_tests COMMAND test_core)
```

---

## Required Commands

| Command | Purpose |
|---------|---------|
| `cmake -S . -B build` | Configure the build |
| `cmake --build build` | Compile the project |
| `ctest --test-dir build` | Run all tests |

**Full workflow:**

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
ctest --test-dir build --output-on-failure
```

---

## Optional Commands

| Command / Tool | Purpose |
|----------------|---------|
| `clang-format` | Auto-format code (use `-i` for in-place) |
| `clang-tidy` | Static analysis (see `.clang-tidy` config below) |
| Address Sanitizer (`-fsanitize=address`) | Detect memory errors at runtime |
| Undefined Behavior Sanitizer (`-fsanitize=undefined`) | Catch UB at runtime |

### Sanitizer Build

```bash
cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer"
cmake --build build-asan
ctest --test-dir build-asan
```

### `.clang-format`

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
ColumnLimit: 100
AllowShortFunctionsOnASingleLine: Empty
BreakBeforeBraces: Attach
SortIncludes: true
```

### `.clang-tidy`

```yaml
Checks: >
  bugprone-*,
  cppcoreguidelines-*,
  modernize-*,
  performance-*,
  portability-*,
  readability-*,
  -modernize-use-trailing-return-type,
  -readability-named-parameter
```

---

## Code Style Rules

### Language Standard

- **Use C++20 or C++23** unless a specific constraint requires an older standard.
- Use `std::format` (C++20) over `printf` or string-stream hacks.
- Use designated initializers, concepts, and ranges where they clarify intent.

### RAII Everywhere

- **Every resource acquisition goes through a constructor. Every release goes through a destructor.**
- Use `std::ifstream` / `std::ofstream`, not `fopen`/`fclose`.
- Use `std::lock_guard` / `std::unique_lock`, not raw `mutex.lock()`/`unlock()`.
- Use `std::vector`, not `new[]`/`delete[]`.

### Smart Pointers

- **Avoid raw owning pointers.** Use:

| Situation | Type |
|-----------|------|
| Unique ownership | `std::unique_ptr<T>` |
| Shared ownership | `std::shared_ptr<T>` |
| Non-owning observer | `T*` (raw pointer OK, but document) |
| Optional ownership | `std::unique_ptr<T>` with null state |

```cpp
// Good — unique ownership
auto resource = std::make_unique<Resource>(args);

// Good — shared ownership
auto shared = std::make_shared<Resource>(args);

// Good — non-owning observer (does not manage lifetime)
void process(const Widget* widget);   // may be null
void process(const Widget& widget);   // must exist
```

### Value Semantics

- **Prefer value semantics** when ownership isn't shared:

```cpp
// Good — pass by const reference (no copy)
void analyze(const std::vector<double>& data);

// Good — return by value (NRVO / move elision)
std::vector<double> compute_result();

// Good — take by value if you need a copy anyway
void store(std::string name) { this->name_ = std::move(name); }
```

### Modern Standard Library Types

- Use `std::optional<T>` for values that may be absent:

```cpp
std::optional<int> parse_int(const std::string& s) {
    try {
        return std::stoi(s);
    } catch (...) {
        return std::nullopt;
    }
}
```

- Use `std::variant<T, U, ...>` for sum types:

```cpp
using Value = std::variant<int, double, std::string>;
Value v = 42;
std::visit([](auto&& arg) { std::cout << arg; }, v);
```

- Use `std::expected<T, E>` (C++23) for fallible operations:

```cpp
std::expected<int, Error> divide(int a, int b) {
    if (b == 0) return std::unexpected(Error::DivideByZero);
    return a / b;
}
```

### Avoid Undefined Behavior

- **Bounds-check** with `.at()` when the index might be invalid.
- **Initialize all variables.** No uninitialized reads.
- **No dangling references** — don't return references to local variables.
- **No strict aliasing violations.** Use `std::bit_cast` (C++20) instead of `reinterpret_cast` for type punning.

### Const-Correctness

- **Use `const` everywhere it applies.** If a function doesn't modify, mark it `const`.

```cpp
class Config {
public:
    explicit Config(std::string path) : path_(std::move(path)) {}

    const std::string& path() const { return path_; }   // const method
    bool is_valid() const { return !path_.empty(); }

private:
    std::string path_;
};
```

### Testing

- **Use a single-header framework** like `doctest` or `Catch2` for fast compile times.
- **Or `GoogleTest`** for larger projects needing fixtures and parameterized tests.

```cpp
// tests/test_core.cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include <doctest/doctest.h>
#include <myproject/core.hpp>

TEST_CASE("adding numbers") {
    CHECK(add(2, 3) == 5);
    CHECK(add(-1, 1) == 0);
}

TEST_CASE("division by zero") {
    CHECK_THROWS_AS(divide(10, 0), std::invalid_argument);
}
```

### Macros

- **Avoid macro-heavy design.** Use `constexpr`, templates, `enum class`, and `inline` functions instead.
- If you must use a macro, `#undef` it as soon as possible and `UPPER_CASE` name it.

### Templates

- **Avoid premature templates.** Don't template a function until you have at least two concrete use cases.
- Prefer `auto` parameters (C++20 abbreviated templates) for simple cases:

```cpp
// Good — simple, C++20
template<typename T>
auto square(T x) { return x * x; }

// Even simpler — abbreviated function template (C++20)
auto square(auto x) { return x * x; }

// Bad — overly complex for one use case
template<typename T, typename Alloc, typename Compare>
class OverEngineeredContainer { ... };
```

### Explicit Performance and Memory Tradeoffs

- If you use `std::move`, `reserve`, pre-allocation, or pointer arithmetic, add a comment explaining why:

```cpp
// Pre-allocate to avoid repeated reallocations during large input processing
result.reserve(expected_size);

// Using std::move to avoid copying the large buffer into the member
this->buffer_ = std::move(buffer);
```

---

## Security Rules

1. **Never use `strcpy`, `sprintf`, `gets`, or unbounded array access.** Use `std::string`, `std::vector`, `std::format`.
2. **Bounds-check all external input.** Use `.at()` or explicit size checks.
3. **No `reinterpret_cast` on user data.**
4. **Initialize every variable.** Even `int x = 0;`.
5. **Stack buffers are bounded.** No `char buf[1024]` for arbitrary input.
6. **Run with sanitizers** in CI: AddressSanitizer + UndefinedBehaviorSanitizer.
7. **Minimize dependencies** — each is a potential vulnerability vector.

---

## Testing Rules

- **Unit tests** — in `tests/` directory, one test executable per module.
- **Test the public API.** Don't test implementation details.
- **Use `TEST_CASE` or `TEST_F` with descriptive names.**
- **Check edge cases:** empty input, maximum values, null/optional states.
- **Run `ctest` in CI** with both normal and sanitizer builds.

---

## Common Mistakes to Avoid

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Raw `new` / `delete` | Memory leaks, double-free, exception-unsafe | `std::unique_ptr`, `std::vector`, `std::make_shared` |
| `std::shared_ptr` for everything | Unnecessary overhead, unclear ownership | `std::unique_ptr` or value semantics |
| Returning `const Foo&` from a temporary | Dangling reference | Return by value |
| `using namespace std;` in headers | Namespace pollution | Fully qualify or use `using std::vector;` in `.cpp` |
| C-style arrays (`int arr[10]`) | No bounds checking, no size info | `std::array` (fixed) or `std::vector` (dynamic) |
| `NULL` or `0` for null pointers | Ambiguous, not type-safe | `nullptr` |
| `auto` everywhere | Can hide important type information | Use explicit types when clarity matters |
| `std::endl` everywhere | Flushes buffer every time — slow | Use `'\n'` unless flush is needed |
| Ignoring compiler warnings | Warnings are bugs you haven't met yet | `-Werror` in CI |
| Template metaprogramming for simple problems | Compile-time bloat, unreadable | Use `if constexpr`, concepts, or just runtime code |
| No move constructors on resource classes | Unnecessary copies | Follow Rule of Zero or Rule of Five |

---

## Example: Minimal Correct C++ Project

```
greet/
├── CMakeLists.txt
├── src/
│   ├── main.cpp
│   └── greet.cpp
├── include/
│   └── greet/
│       └── greet.hpp
└── tests/
    ├── CMakeLists.txt
    └── test_greet.cpp
```

**`CMakeLists.txt`**

```cmake
cmake_minimum_required(VERSION 3.20)
project(greet VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(NOT MSVC)
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

add_library(greet_lib src/greet.cpp)
target_include_directories(greet_lib PUBLIC include)

add_executable(greet_app src/main.cpp)
target_link_libraries(greet_app PRIVATE greet_lib)

enable_testing()
add_subdirectory(tests)
```

**`include/greet/greet.hpp`**

```cpp
#pragma once

#include <string>
#include <stdexcept>

namespace greet {

/**
 * Returns a greeting for the given name.
 * @throws std::invalid_argument if name is empty.
 */
std::string greet(const std::string& name);

} // namespace greet
```

**`src/greet.cpp`**

```cpp
#include "greet/greet.hpp"

namespace greet {

std::string greet(const std::string& name) {
    if (name.empty()) {
        throw std::invalid_argument("name must not be empty");
    }
    return std::format("Hello, {}!", name);
}

} // namespace greet
```

**`src/main.cpp`**

```cpp
#include <iostream>
#include <string>
#include "greet/greet.hpp"

int main(int argc, char* argv[]) {
    std::string name = (argc > 1) ? argv[1] : "World";
    try {
        std::cout << greet::greet(name) << '\n';
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << '\n';
        return 1;
    }
    return 0;
}
```

**`tests/CMakeLists.txt`**

```cmake
include(FetchContent)
FetchContent_Declare(
    doctest
    GIT_REPOSITORY https://github.com/doctest/doctest.git
    GIT_TAG v2.4.11
)
FetchContent_MakeAvailable(doctest)

add_executable(test_greet test_greet.cpp)
target_link_libraries(test_greet PRIVATE greet_lib doctest::doctest)
add_test(NAME greet_tests COMMAND test_greet)
```

**`tests/test_greet.cpp`**

```cpp
#define DOCTEST_CONFIG_IMPLEMENT_WITH_MAIN
#include <doctest/doctest.h>
#include "greet/greet.hpp"

TEST_CASE("greet returns correct greeting") {
    CHECK(greet::greet("Ada") == "Hello, Ada!");
    CHECK(greet::greet("World") == "Hello, World!");
}

TEST_CASE("greet throws on empty name") {
    CHECK_THROWS_AS(greet::greet(""), std::invalid_argument);
}
```

**Run the example:**

```bash
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
./build/greet_app "Ada"
```

---

## Quick Reference

```bash
# Configure and build
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel

# Run tests
ctest --test-dir build --output-on-failure

# Format all sources
find src include tests -name '*.cpp' -o -name '*.hpp' | xargs clang-format -i

# Run static analysis
clang-tidy src/*.cpp -- -Iinclude

# Sanitizer build for debugging
cmake -S . -B build-debug -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_CXX_FLAGS="-fsanitize=address,undefined -fno-omit-frame-pointer"
cmake --build build-debug
ctest --test-dir build-debug

# Clean rebuild
cmake --build build --target clean
cmake --build build
```
