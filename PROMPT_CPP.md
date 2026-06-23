# C++-Specific Claude Code Prompt

Append this prompt after `PROMPT_UNIVERSAL.md` for all C++ projects. These rules supplement -- not replace -- the universal workflow.

---

## C++ Build Commands

Use these commands throughout the workflow:

| Command | Purpose | When to run |
|---------|---------|-------------|
| `cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug` | Configure debug build | Initial setup, after CMake changes |
| `cmake -S . -B build -DCMAKE_BUILD_TYPE=Release` | Configure release build | Before benchmarking |
| `cmake --build build` | Compile project | After code changes |
| `cmake --build build --parallel` | Parallel build | Faster compilation |
| `ctest --test-dir build` | Run test suite | After implementation, must pass |
| `ctest --test-dir build --output-on-failure` | Run tests with output on failure | Debugging |
| `cmake --build build --target format` | Run clang-format (if configured) | Before every commit |
| `cmake --build build --target tidy` | Run clang-tidy (if configured) | Before every commit |
| `./build/src/executable` | Run executable | Manual verification |

### Optional Sanitizer Builds

```bash
# Address Sanitizer (detects use-after-free, buffer overflow, memory leaks)
cmake -S . -B build-asan -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=address -fno-omit-frame-pointer"
cmake --build build-asan
ctest --test-dir build-asan

# Undefined Behavior Sanitizer
cmake -S . -B build-ubsan -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=undefined"
cmake --build build-ubsan
ctest --test-dir build-ubsan

# Thread Sanitizer (for threaded code)
cmake -S . -B build-tsan -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS="-fsanitize=thread"
cmake --build build-tsan
ctest --test-dir build-tsan
```

Run sanitizer builds before final delivery if the project uses manual memory management or threading.

---

## C++ Code Rules

### Language Standard
- Use **C++20** as the baseline. Use **C++23** features when the toolchain supports them (GCC 13+, Clang 17+, MSVC 2022 17.8+).
- Prefer standard library features over Boost. Use Boost only for ASIO or serialization if justified.
- Use designated initializers, concepts, ranges, `std::format`, and modules where appropriate.

### RAII and Resource Management
- **RAII everywhere.** Every resource (memory, file handles, locks, network sockets) must be owned by an object.
- Use `std::unique_ptr` for exclusive ownership. Use `std::shared_ptr` for shared ownership (rare).
- **No raw owning pointers.** A raw pointer (`T*`) is a non-owning view only.
- Use `std::make_unique<T>()` and `std::make_shared<T>()` -- never raw `new`/`delete`.
- Use `std::vector<T>` as the default container. Use `std::array<T, N>` for fixed-size.

### Value Semantics
- Prefer value semantics over reference semantics. Pass by value when copying is cheap, by `const T&` when expensive.
- Return values by value. Rely on NRVO/copy elision. Do not use output parameters unless necessary.
- Use `std::move` explicitly at transfer points. Do not `std::move` from a variable you will use later.

### Modern C++ Types
- Use `std::optional<T>` for values that may be absent. No sentinel values.
- Use `std::variant<T1, T2, ...>` for sum types. No raw unions.
- Use `std::expected<T, E>` (C++23) or `tl::expected` (C++20) for fallible operations.
- Use `std::span<T>` for non-owning array views. Use `std::string_view` for string parameters.
- Use `std::tuple` only for internal implementation. Prefer named structs for public APIs.

### Error Handling
- Use exceptions for exceptional conditions. Use `std::expected` or `std::optional` for expected failures.
- Throw standard exception types: `std::invalid_argument`, `std::out_of_range`, `std::runtime_error`.
- Custom exceptions inherit from `std::exception`:
  ```cpp
  class parse_error : public std::runtime_error {
  public:
      explicit parse_error(const std::string& msg) : std::runtime_error(msg) {}
  };
  ```
- Exception messages must be actionable: state what failed and why.
- Document `noexcept` on functions that guarantee not to throw.

### Const Correctness
- Use `const` everywhere it applies. `const` member functions, `const` references, `const` iterators.
- Use `constexpr` for compile-time computable values and functions.
- Use `consteval` (C++20) for functions that must execute at compile time.
- Mark variables `const` by default. Remove `const` only when mutation is needed.

### Undefined Behavior
- **No undefined behavior.** No null pointer dereference, no buffer overflow, no use-after-free, no strict aliasing violation.
- Check preconditions: `assert(condition)` for internal invariants (compiled out in Release).
- Use `.at()` instead of `operator[]` when bounds are uncertain.
- Initialize all variables. No uninitialized reads.
- Use `std::size_t` for sizes and indices. Be careful with signed/unsigned comparisons.

### Templates
- Avoid premature templates. Start concrete, abstract when duplication appears.
- Use concepts (C++20) to constrain templates: `template<std::integral T>` not `template<typename T>` with SFINAE.
- Keep template code readable. Complex metaprogramming requires extensive documentation.
- Prefer function overloading and `if constexpr` over SFINAE.

### Macros
- **No macro-heavy design.** Macros are for include guards and conditional compilation only.
- Use `enum class` instead of `#define` constants.
- Use `inline constexpr` variables instead of macro constants.
- Use templates instead of macro-based generics.
- If a macro is necessary, document exactly why and scope it minimally.

### Performance
- Profile before optimizing. Use `perf`, `vtune`, or `gprof`.
- Use `[[likely]]` / `[[unlikely]]` (C++20) for hot path annotations only with evidence.
- Use `std::string_view` to avoid string copies in parsing.
- Use `reserve()` on containers when size is known.
- Consider cache locality in data structure layout (Structure of Arrays vs Array of Structures).
- Document memory/performance tradeoffs explicitly.

---

## C++ Testing

### Test Framework
- Prefer **Catch2 v3** (header-only or compiled) or **doctest** (lightweight, single-header) for new projects.
- Use **GoogleTest** if the project already uses it or if GoogleMock is needed.
- Test file naming: `test_*.cpp` or `*_test.cpp`. Be consistent.

### Test Organization
```
tests/
  test_main.cpp       -- Catch2 main (if not using default main)
  test_core.cpp       -- Core logic tests
  test_edge_cases.cpp -- Edge case and error tests
  test_utilities.cpp  -- Utility function tests
```

### Test Practices
- Use `SECTION` / `TEST_CASE` (Catch2) or `TEST()` (GoogleTest) for grouping.
- Use descriptive test names: `TEST_CASE("rejects negative deposit amounts")` not `TEST_CASE("test 3")`.
- Test every public function. Test error paths with `REQUIRE_THROWS`.
- Use `REQUIRE` for fatal assertions, `CHECK` for non-fatal (Catch2).
- Use explicit test data. No shared mutable fixtures between tests.
- Use `constexpr` tests where possible to verify compile-time behavior.

### Test Commands
```bash
ctest --test-dir build                    # Run all tests
ctest --test-dir build --output-on-failure # With output on failure
ctest --test-dir build -R test_name       # Run specific test
```

---

## CMakeLists.txt Requirements

Use this CMake configuration as a baseline:

```cmake
cmake_minimum_required(VERSION 3.20)
project(project_name VERSION 0.1.0 LANGUAGES CXX)

# C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Compiler warnings (treat as errors)
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(
        -Wall -Wextra -Wpedantic
        -Wconversion -Wsign-conversion
        -Wshadow -Wnon-virtual-dtor
        -Wold-style-cast -Wcast-align
        -Wunused -Woverloaded-virtual
        -Werror
    )
elseif(MSVC)
    add_compile_options(/W4 /WX)
endif()

# Sanitizer option (Debug builds)
option(ENABLE_ASAN "Enable Address Sanitizer" OFF)
if(ENABLE_ASAN AND CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_compile_options(-fsanitize=address -fno-omit-frame-pointer)
    add_link_options(-fsanitize=address)
endif()

# Library
add_library(project_lib
    src/core.cpp
    src/utils.cpp
)
target_include_directories(project_lib PUBLIC include)

# Executable
add_executable(project_exe src/main.cpp)
target_link_libraries(project_exe PRIVATE project_lib)

# Testing
option(BUILD_TESTS "Build tests" ON)
if(BUILD_TESTS)
    enable_testing()
    find_package(Catch2 3 REQUIRED)
    add_executable(project_tests
        tests/test_core.cpp
        tests/test_edge_cases.cpp
    )
    target_link_libraries(project_tests PRIVATE project_lib Catch2::Catch2WithMain)
    include(Catch)
    catch_discover_tests(project_tests)
endif()

# clang-format target
find_program(CLANG_FORMAT NAMES clang-format)
if(CLANG_FORMAT)
    file(GLOB_RECURSE ALL_SOURCE_FILES src/*.cpp include/*.hpp tests/*.cpp)
    add_custom_target(format
        COMMAND ${CLANG_FORMAT} -i ${ALL_SOURCE_FILES}
        COMMENT "Running clang-format"
    )
endif()
```

---

## C++ Formatting and Linting

### clang-format (.clang-format)

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
ColumnLimit: 100
BreakBeforeBraces: Attach
AllowShortFunctionsOnASingleLine: Empty
SortIncludes: true
IncludeBlocks: Regroup
PointerAlignment: Left
ReferenceAlignment: Left
SpacesInParentheses: false
IndentCaseLabels: false
```

### clang-tidy (.clang-tidy)

```yaml
Checks: >
  bugprone-*,
  cppcoreguidelines-*,
  modernize-*,
  performance-*,
  portability-*,
  readability-*,
  -cppcoreguidelines-avoid-magic-numbers,
  -readability-named-parameter
WarningsAsErrors: '*'
HeaderFilterRegex: '.*'
```

Run both before delivery:
```bash
clang-format -i src/**/*.cpp include/**/*.hpp tests/**/*.cpp
clang-tidy src/**/*.cpp -- -Iinclude
```

---

## C++ Security

- No raw `new`/`delete`. Smart pointers or containers only.
- Use `std::string` and `std::vector` -- no C-style arrays or `char*` buffers.
- Validate all external input at the program boundary (files, network, user input).
- Use `std::optional` or `std::expected` instead of sentinel values that could be misinterpreted.
- No buffer overflows: use `.at()` or iterators with bounds checking when input is untrusted.
- Mark destructors `virtual` in polymorphic base classes.
- No `reinterpret_cast` without justification and safety proof.
- Prefer `enum class` over `enum` to avoid implicit conversions.
- Use `[[nodiscard]]` on functions where ignoring the return value is a bug.
- Run with AddressSanitizer before final delivery if the code manages memory manually.

---

## C++ Project Structure Template

Use this structure unless the project demands otherwise:

```
project_name/
  CMakeLists.txt           -- Build configuration
  .clang-format            -- Formatting rules
  .clang-tidy              -- Linting rules
  cmake/
    FindDependencies.cmake -- Custom find modules (if needed)
  include/
    project/
      core.hpp             -- Public API header
      types.hpp            -- Core type definitions
      utils.hpp            -- Utility functions
  src/
    core.cpp               -- Core implementation
    utils.cpp              -- Utility implementation
    main.cpp               -- Application entry point (if binary)
  tests/
    test_main.cpp          -- Test runner (if needed)
    test_core.cpp          -- Core logic tests
    test_edge_cases.cpp    -- Edge case tests
    test_utilities.cpp     -- Utility tests
  benchmarks/              -- (if performance matters)
    bench_core.cpp
  examples/
    basic_example.cpp
  README.md                -- Build, test, usage instructions
  LICENSE
  .gitignore
```

---

## C++ Review Checklist

Before marking a C++ project complete, verify every item:

### Code Quality
- [ ] Builds cleanly: `cmake --build build` with zero warnings
- [ ] `clang-format` produces no changes (or code is consistently formatted)
- [ ] `clang-tidy` passes (if configured)
- [ ] No raw `new`/`delete`
- [ ] No raw owning pointers
- [ ] `const` correctness throughout
- [ ] All variables initialized
- [ ] No macros except include guards/conditional compilation
- [ ] RAII for all resources

### Modern C++
- [ ] C++20 or C++23 features used appropriately
- [ ] `std::optional` / `std::variant` / `std::expected` used instead of sentinels
- [ ] `std::span` / `std::string_view` for non-owning views
- [ ] Concepts used to constrain templates (not SFINAE)
- [ ] `constexpr` / `consteval` used for compile-time computation
- [ ] `[[nodiscard]]` on important return values

### Testing
- [ ] `ctest --test-dir build` passes all tests
- [ ] Every public function tested
- [ ] Error paths tested with `REQUIRE_THROWS`
- [ ] Edge cases tested (empty input, zero, max values, boundary conditions)
- [ ] No undefined behavior (verified by UBSan if applicable)
- [ ] Memory safety verified (AddressSanitizer if applicable)

### Performance
- [ ] Benchmarks exist if performance matters
- [ ] No premature optimization without profiling data
- [ ] Container `reserve()` used where size is known
- [ ] Pass-by-value vs pass-by-reference is intentional

### Security
- [ ] All external input validated at boundaries
- [ ] No buffer overflows possible
- [ ] No use-after-free or memory leaks (ASan clean)
- [ ] `reinterpret_cast` absent or justified
- [ ] No secrets in source code

### Documentation
- [ ] README includes: build instructions, dependencies, test/run commands
- [ ] Public API documented in headers (doc comments)
- [ ] Examples compile and run
- [ ] Complex algorithms explained with comments

---

## C++-Specific Decision Guide

| Decision | Prefer | Avoid |
|----------|--------|-------|
| Strings | `std::string` / `std::string_view` | `char*`, C-string functions |
| Containers | `std::vector`, `std::array` | C arrays, raw `new[]` |
| Pointers | `std::unique_ptr`, `std::shared_ptr` | Raw owning pointers |
| Optional values | `std::optional<T>` | Null pointers, sentinel values |
| Error handling | Exceptions / `std::expected` | Error codes, boolean returns |
| Views | `std::span<T>`, `std::string_view` | Pointer + length pairs |
| Type variants | `std::variant` | Raw unions, `void*` + type tag |
| Math | `<cmath>`, `<numbers>` (C++20) | Hand-rolled math |
| I/O | `<fstream>`, `<iostream>` | `stdio.h` (unless justified) |
| Random | `<random>` | `rand()` |
| Time | `<chrono>` | `time.h`, `gettimeofday` |
| Formatting | `std::format` (C++20) | `printf`, `sprintf` |
| File system | `<filesystem>` (C++17) | Manual path manipulation |
| Threads | `std::jthread` (C++20), `std::async` | Raw `pthread`, manual thread management |
| Testing | Catch2 v3 or doctest | Ad-hoc testing |
| JSON | `nlohmann/json` | Manual JSON parsing |
| CLI args | `CLI11` or `lyra` | Manual argv parsing |
