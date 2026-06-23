# Python-Specific Claude Code Prompt

Append this prompt after `PROMPT_UNIVERSAL.md` for all Python projects. These rules supplement -- not replace -- the universal workflow.

---

## Python Build Commands

Use these commands throughout the workflow:

| Command | Purpose | When to run |
|---------|---------|-------------|
| `python -m venv .venv` | Create isolated environment | Project initialization |
| `pip install -e ".[dev]"` | Install package + dev tools (editable) | After project setup |
| `ruff format .` | Auto-format code | Before every commit |
| `ruff check .` | Lint: style, bugs, import order | Before every commit, must pass |
| `ruff check . --fix` | Auto-fix safe lint findings | During cleanup |
| `mypy src` | Static type check (strict) | After implementation, must pass |
| `pytest` | Run all tests | After implementation, must pass |
| `pytest --cov=src --cov-report=term-missing` | Tests + coverage gaps | Review step |
| `python -m my_project` | Run the package | Manual verification |
| `python -m cProfile -s cumtime -m my_project` | Profile hot paths | Step 13 (Optimization Pass) |
| `pip-audit` | Dependency CVE scan | Security review |

---

## Python Code Rules

### Types and Boundaries
- Annotate every public function: parameter types and return type.
- Use modern syntax: `list[int]`, `dict[str, T]`, `X | None` (not `List`, `Optional`).
- `mypy --strict` must pass. No implicit `Any` at module edges.
- Model structured data with `@dataclass` (internal) or `pydantic.BaseModel` (validated boundary), never bare `dict`.
- Use `enum.Enum` for fixed sets of values; use `pathlib.Path` for paths.

### Error Handling
- Raise the narrowest specific exception; define a small module-level exception hierarchy for your own errors.
- Never `except:` or `except Exception: pass`. Catch what you can handle; otherwise let it propagate.
- Preserve the cause chain with `raise NewError(...) from original`.
- Validate untrusted input at the boundary; trust it internally.
- Error messages are actionable: what went wrong and what the caller can do.

### Structure
- Prefer functions to classes. Introduce a class only when state and behavior travel together.
- No mutable default arguments. Default to `None`, construct inside the function.
- No import-time side effects. Work goes behind functions or `if __name__ == "__main__":`.
- Absolute imports from the package root; no `import *`.
- Keep functions single-purpose and nameable.

### Concurrency
- I/O-bound: `asyncio` or threads. CPU-bound: `multiprocessing` / `ProcessPoolExecutor` (the GIL blocks parallel CPU work on threads).
- Never block the event loop; wrap blocking calls with `asyncio.to_thread`.
- Guard shared mutable state with a `Lock` or pass work through a `queue.Queue`.
- Do not add `asyncio` unless there is real concurrent I/O.

### Performance
- Profile first (`cProfile`, `perf_counter`), optimize second. No optimization without a measurement.
- Prefer built-ins, comprehensions, and generators (C-speed, low memory).
- Vectorize numeric hot loops with `numpy` instead of Python-level loops.
- Use `functools.lru_cache` for pure, repeatedly-called functions.

---

## Python Security

- **No `eval`/`exec` on dynamic input. No `pickle.loads` on untrusted data.** Use `json` for untrusted serialization.
- Run `pip-audit` before final delivery; address or document every finding.
- No secrets in source. Load from environment or a secret manager. `.env` belongs in `.gitignore`.
- Validate all external input at the boundary (files, network, env, CLI).
- `subprocess` calls pass an argument list with `shell=False`; never concatenate a shell string.
- Canonicalize user-derived paths (`Path.resolve()`) and check them against an allow-listed base directory.

---

## Python Testing

### Test Organization
```
src/
  my_project/
    core.py        -- pure logic
    io.py          -- boundary I/O
tests/
  conftest.py      -- shared fixtures
  test_core.py     -- unit tests for core
  test_io.py       -- tests for the I/O boundary (mocked)
```

### Unit Tests
- One behavior per test; name `test_<behavior>` (e.g. `test_rejects_empty_name`).
- Every public function: at least one success case and one error/edge case.
- Use `pytest.raises` to assert on exceptions, asserting the type (and message when it matters).
- Use `@pytest.mark.parametrize` for matrices instead of copy-pasted tests.

### Fixtures and Isolation
- Shared setup goes in `conftest.py` fixtures.
- Use `tmp_path` for filesystem work and `monkeypatch` for env vars / attributes.
- A unit test touches no real network and no real disk outside `tmp_path`.

### Coverage and Doctests
- Aim for meaningful coverage of branches, not a percentage for its own sake; show gaps with `--cov-report=term-missing`.
- Public functions may carry doctest examples in their docstring; run them with `pytest --doctest-modules`.

---

## Python Project Structure Template

Use this structure unless the project demands otherwise:

```
project_name/
  pyproject.toml        -- deps, metadata, ruff/mypy/pytest config (single source of truth)
  README.md             -- what it does, install, test, run
  .gitignore            -- includes .venv/ and .env
  src/
    project_name/
      __init__.py       -- version, public re-exports
      __main__.py       -- CLI entry point (python -m project_name)
      config.py         -- configuration parsing + validation
      errors.py         -- exception hierarchy
      core.py           -- pure business logic (no I/O)
      io.py             -- file/network/database I/O
  tests/
    conftest.py         -- shared fixtures
    test_core.py
    test_config.py
```

### pyproject.toml Requirements

```toml
[project]
name = "project_name"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    # Every dependency must be justified in the Architecture Plan
]

[project.optional-dependencies]
dev = ["ruff", "mypy", "pytest", "pytest-cov"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

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

## Python Review Checklist

Before marking a Python project complete, verify every item:

### Code Quality
- [ ] `ruff format .` produces no changes
- [ ] `ruff check .` passes cleanly
- [ ] `mypy src` passes under `strict`
- [ ] All public functions have type hints
- [ ] No bare `except` and no silently-swallowed exceptions
- [ ] No mutable default arguments
- [ ] Structured data uses dataclasses/pydantic, not bare dicts
- [ ] No import-time side effects

### Testing
- [ ] `pytest` passes (all tests)
- [ ] Every public function has at least one test
- [ ] Error and edge cases are tested with `pytest.raises`
- [ ] Tests use `tmp_path`/`monkeypatch` — no real disk/network
- [ ] Coverage gaps reviewed (`--cov-report=term-missing`)

### Security
- [ ] No `eval`/`exec`/`pickle` on untrusted input
- [ ] `pip-audit` passes or findings are documented
- [ ] No secrets in source; `.env` in `.gitignore`
- [ ] External input validated at the boundary
- [ ] `subprocess` uses argument lists, `shell=False`

### Documentation
- [ ] README: what it does, how to install, how to test, how to run
- [ ] Public functions have docstrings (with examples where useful)
- [ ] `python -m project_name` runs and behaves as documented

---

## Python-Specific Decision Guide

| Decision | Prefer | Avoid |
|----------|--------|-------|
| Dependency / venv management | `uv` or stdlib `venv` + `pyproject.toml` | global `pip install`, `setup.py` |
| Structured internal data | `@dataclass(slots=True)` | bare `dict` |
| Validated boundary data | `pydantic.BaseModel` | hand-rolled validation |
| Lint + format | `ruff` (both) | `flake8` + `black` + `isort` separately |
| Type checking | `mypy --strict` | no type checking |
| Testing | `pytest` + fixtures | `unittest` boilerplate |
| CLI parsing | `argparse` (stdlib) or `typer` | `sys.argv` hand-parsing |
| HTTP client | `httpx` (sync/async) | raw `urllib` for complex calls |
| Web/API framework | `FastAPI` (typed, async) | `Flask` for new typed APIs |
| Config | `pydantic-settings` / env vars | hardcoded constants |
| Paths | `pathlib.Path` | `os.path` string joins |
| CPU-bound parallelism | `multiprocessing` / `ProcessPoolExecutor` | threads (GIL) |
| I/O concurrency | `asyncio` / threads | blocking calls in async |
| Serialization (untrusted) | `json` | `pickle` |
| Logging | `logging` (structured) | `print` debugging |
