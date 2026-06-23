# Python Best Practices for Claude Code Agent Swarm

> One-sentence rule: **Write code that is obvious on first read, typed at its boundaries, and proven by a failing test before it is proven by a passing one.**

---

## Philosophy

- **Readability is the feature** — Python's value is that humans (and agents) understand it fast; never trade clarity for cleverness.
- **Types at the boundaries** — dynamic typing is fine inside a function; public functions, data models, and module edges carry explicit type hints and pass `mypy`.
- **Batteries, then ecosystem** — reach for the standard library first; add a dependency only when it earns its place (each one is a trust boundary).
- **Explicit over implicit** — no hidden state, no import-time side effects, no mutable default arguments. The program does what it reads like it does.

---

## Project Setup

### New Project (uv — recommended)

```bash
uv init project_name
cd project_name
uv add --dev ruff mypy pytest
```

### New Project (stdlib venv — no extra tooling)

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

### Project Layout (src layout — preferred)

Use the `src/` layout so tests run against the installed package, not the working directory:

```
my-project/
├── pyproject.toml          # single source of truth: deps, tooling, metadata
├── src/
│   └── my_project/
│       ├── __init__.py
│       ├── __main__.py     # `python -m my_project`
│       ├── config.py
│       ├── core.py
│       └── errors.py
└── tests/
    ├── test_core.py
    └── conftest.py
```

**Minimal `pyproject.toml`:**

```toml
[project]
name = "my_project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = []

[project.optional-dependencies]
dev = ["ruff", "mypy", "pytest", "pytest-cov"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM", "RUF"]

[tool.mypy]
python_version = "3.11"
strict = true

[tool.pytest.ini_options]
addopts = "-q"
testpaths = ["tests"]
```

---

## Required Commands

| Command | Purpose |
|---------|---------|
| `python -m venv .venv` | Create an isolated virtual environment |
| `pip install -e ".[dev]"` | Install package + dev tools in editable mode |
| `ruff format .` | Auto-format all code |
| `ruff check .` | Lint (style, bugs, imports) |
| `mypy src` | Static type check |
| `pytest` | Run all tests |

**Always run the following before committing:**

```bash
ruff format . && ruff check . && mypy src && pytest
```

---

## Optional Commands

| Command | Purpose |
|---------|---------|
| `pytest --cov=src --cov-report=term-missing` | Test run with coverage gaps shown |
| `ruff check . --fix` | Auto-fix lint findings that are safe to fix |
| `pip-audit` | Scan installed dependencies for known CVEs |
| `python -X dev -m my_project` | Run with dev-mode warnings (resource leaks, deprecations) |

---

## Code Style Rules

### Naming

| Item | Convention | Example |
|------|-----------|---------|
| Functions, variables, modules | `snake_case` | `compute_total`, `user_name` |
| Classes, exceptions | `PascalCase` | `HttpRequest`, `ParserError` |
| Constants | `SCREAMING_SNAKE_CASE` | `MAX_BUFFER_SIZE` |
| "Private" names | leading underscore | `_internal_cache` |
| Type variables | `PascalCase`, short | `T`, `KT`, `VT` |
| Test functions | `test_<behavior>` | `test_rejects_empty_name` |

### Type Hints

- **Annotate every public function** — parameters and return type. Internal helpers may omit hints where the type is obvious.
- **Use modern syntax** (3.10+): `list[int]`, `dict[str, int]`, `X | None` instead of `List`, `Dict`, `Optional[X]`.
- **`mypy --strict` must pass.** No untyped `def`s at module boundaries, no implicit `Any` returns.
- **Type the data, not just the functions** — use `@dataclass` or `pydantic.BaseModel` for structured data instead of bare dicts.

```python
from dataclasses import dataclass

@dataclass(frozen=True, slots=True)
class Config:
    name: str
    port: int = 8080
```

### Error Handling

- **Raise specific exceptions**, never bare `Exception`. Define a small module-level hierarchy for your own errors:

```python
class ConfigError(Exception):
    """Base for all configuration problems."""

class MissingFieldError(ConfigError):
    def __init__(self, field: str) -> None:
        super().__init__(f"missing required field: {field}")
        self.field = field
```

- **Never** `except:` or `except Exception: pass` — catch the narrowest type you can handle, and either handle it or re-raise.
- **Add context with `raise ... from`** so the cause chain is preserved:

```python
try:
    data = json.loads(text)
except json.JSONDecodeError as e:
    raise ConfigError("config is not valid JSON") from e
```

- **Validate at the boundary** (user input, file contents, network, env), then trust the data inside.

### Data & Serialization

- Use `json` from the stdlib for simple cases; reach for `pydantic` when you need validation, coercion, and schema at the same time.
- Prefer `dataclasses` for internal records that don't need validation; prefer `pydantic.BaseModel` for anything crossing a trust boundary.
- Use `pathlib.Path` for filesystem paths — never string concatenation.
- Use `enum.Enum` instead of stringly-typed magic values.

### Concurrency

- **Pick the right model:**
  - I/O-bound (network, disk, DB) → `asyncio` or threads.
  - CPU-bound → `multiprocessing` or a `ProcessPoolExecutor` (the GIL blocks threads from parallel CPU work).
- **Do not mix blocking calls into async code** — wrap them with `asyncio.to_thread(...)`.
- **Never share mutable state across threads without a lock** (`threading.Lock`) or a queue (`queue.Queue`).
- Don't introduce `asyncio` for "future-proofing"; add it when there is real concurrent I/O.

### Functions and Classes

- **Functions first** — reach for a class only when you have state + behavior that travel together. A class with one method should be a function.
- **Keep functions small and single-purpose.** If you can't name it clearly, it does too much.
- **No mutable default arguments** (`def f(items=[])` is a bug). Use `None` and create inside:

```python
def append_to(value: int, items: list[int] | None = None) -> list[int]:
    items = items if items is not None else []
    items.append(value)
    return items
```

### Performance

- **Profile before optimizing** — `python -m cProfile -s cumtime` or `time.perf_counter()` around the hot path. Never optimize on a hunch.
- Prefer built-in data structures and comprehensions; they run in C.
- Use generators for large sequences to avoid materializing everything in memory.
- For numeric hot loops, vectorize with `numpy` rather than hand-rolling Python loops.

---

## Security Rules

1. **Never `eval()` / `exec()` on untrusted input**, and never `pickle.loads()` data you did not produce. Use `json` for untrusted data.
2. **Validate all external input** — files, network, environment variables, CLI args — before use. Whitelist over blacklist.
3. **No secrets in source.** Load from environment or a secret manager. Keep `.env` in `.gitignore`.
4. **Run `pip-audit` (or `safety`)** to check dependencies for known CVEs; pin versions with a committed lockfile.
5. **Never build shell commands by string concatenation.** Use `subprocess.run([...], shell=False)` with an argument list.
6. **Canonicalize and bound file paths** built from user input (`Path.resolve()` + an allow-list base) to prevent traversal.

---

## Testing Rules

- **Tests live in `tests/`**, named `test_*.py`, with functions named `test_<behavior>`. Mirror the package structure.
- **Test the public API as a consumer would** — import from the installed package, not by reaching into private modules.
- **Every public function gets at least one success case and one failure/edge case.**
- **Use fixtures (`conftest.py`) for shared setup**; use `tmp_path` for files and `monkeypatch` for env/attributes — never touch real disk or real network in a unit test.
- **Parametrize** instead of copy-pasting near-identical tests:

```python
import pytest
from my_project.core import greet

@pytest.mark.parametrize("name,expected", [
    ("Ada", "Hello, Ada!"),
    ("Bo", "Hello, Bo!"),
])
def test_greet_returns_greeting(name: str, expected: str) -> None:
    assert greet(name) == expected

def test_greet_rejects_empty() -> None:
    with pytest.raises(ValueError):
        greet("")
```

---

## Project Structure Template

```
my_project/
├── pyproject.toml          # deps, ruff, mypy, pytest config — one file
├── README.md               # what it does, install, test, run
├── .gitignore              # includes .venv/ and .env
├── src/
│   └── my_project/
│       ├── __init__.py     # package version, public re-exports
│       ├── __main__.py     # CLI entry: `python -m my_project`
│       ├── config.py       # configuration parsing + validation
│       ├── errors.py       # exception hierarchy
│       ├── core.py         # pure business logic (no I/O)
│       └── io.py           # file / network / DB boundary
└── tests/
    ├── conftest.py         # shared fixtures
    ├── test_core.py
    └── test_config.py
```

---

## Common Mistakes to Avoid

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Mutable default argument (`def f(x=[])`) | Shared across calls; classic bug | Default `None`, create inside |
| `except:` / `except Exception: pass` | Hides real errors | Catch narrowly; handle or re-raise |
| `eval`/`exec`/`pickle` on untrusted input | Arbitrary code execution | Use `json`; parse explicitly |
| Bare `dict` for structured data | No types, typos go unnoticed | `@dataclass` or `pydantic` |
| String concatenation for paths | Breaks cross-platform, traversal risk | `pathlib.Path` |
| Threads for CPU-bound work | GIL serializes them | `multiprocessing` / `ProcessPoolExecutor` |
| No type hints at module edges | `mypy` can't help; callers guess | Annotate public signatures |
| Import-time side effects | Surprises on `import` | Put work behind functions / `__main__` |
| `from module import *` | Pollutes namespace, hides origins | Import names explicitly |
| Catching then logging *and* swallowing | Error vanishes silently | Re-raise or convert to a typed error |
| Relative imports across the project | Fragile, breaks on move | Absolute imports from the package root |

---

## Example: Minimal Correct Python Project

```
greet/
├── pyproject.toml
├── src/
│   └── greet/
│       ├── __init__.py
│       └── core.py
└── tests/
    └── test_core.py
```

**`pyproject.toml`**

```toml
[project]
name = "greet"
version = "0.1.0"
requires-python = ">=3.11"

[project.optional-dependencies]
dev = ["ruff", "mypy", "pytest"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.mypy]
strict = true
```

**`src/greet/core.py`**

```python
"""Greeting logic with no I/O — pure and testable."""


def greet(name: str) -> str:
    """Return a greeting for ``name``.

    >>> greet("Ada")
    'Hello, Ada!'

    Raises:
        ValueError: if ``name`` is empty.
    """
    if not name:
        raise ValueError("name must not be empty")
    return f"Hello, {name}!"
```

**`src/greet/__init__.py`**

```python
from greet.core import greet

__all__ = ["greet"]
__version__ = "0.1.0"
```

**`tests/test_core.py`**

```python
import pytest
from greet import greet


def test_greet_ok() -> None:
    assert greet("Ada") == "Hello, Ada!"


def test_greet_empty() -> None:
    with pytest.raises(ValueError):
        greet("")
```

**Run the example:**

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
ruff format . && ruff check . && mypy src && pytest
```

---

## Quick Reference

```bash
# Full pre-commit check
ruff format . && ruff check . && mypy src && pytest

# Test with coverage gaps
pytest --cov=src --cov-report=term-missing

# Run the package as a CLI
python -m my_project

# Profile a hot path
python -m cProfile -s cumtime -m my_project

# Audit dependencies for CVEs
pip-audit

# Dev mode (surfaces resource leaks + deprecations)
python -X dev -m my_project
```
