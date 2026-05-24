# Code Standards & Engineering Contract

## 1. Type Safety & Data Modeling
- **Explicit Type Hinting:** All function signatures, variable assignments, and class definitions must utilize strict Python type hinting. The use of `any` or `Any` is structurally prohibited unless passing unparsed raw third-party payloads.
- **Structural Typing via Protocols:** Use `typing.Protocol` to define structural interfaces and behaviors rather than relying on heavy abstract base classes (`abc.ABC`).
- **Pydantic V2 Enforcement:** Every internal data boundary, configuration schema, and external API payload must be modeled using Pydantic V2. 
  - Mutable attributes must be restricted; prefer `frozen=True` on models unless mutation is structurally required.
  - Use explicit `Field` constraints (`Field(gt=0)`, `Field(max_length=X)`) instead of post-init validation blocks.
- **No Raw Dictionaries:** Passing unvalidated raw `dict` objects between application layers is forbidden. Use strongly-typed Pydantic models, `NamedTuple`, or `dataclasses`.

## 2. Structural Metrics & Code Density
- **The 30-Line Ceiling:** No function, method, or coroutine may exceed 30 lines of executable code (excluding docstrings and type definitions). If a function crosses this threshold, it must be refactored into smaller, single-responsibility units.
- **Cyclomatic Complexity Limit:** Maximize logical simplicity. Avoid deeply nested conditional blocks (`if/else`). Prefer guard clauses, early returns, and dictionary/match-case dispatch maps. Nesting depth must not exceed 3 levels.

## 3. Error Handling & Robustness Framework
- **Zero Swallowed Exceptions:** Empty `except: pass` blocks and generic, un-logged `except Exception:` catches are strictly banned.
- **Explicit Propagation:** If an exception is caught, it must either be enriched and re-raised (`raise NewException("context") from err`), or piped directly into a structured logger containing the full stack trace and execution state.
- **Domain-Specific Exceptions:** Define explicit exception hierarchies per module. Do not leak low-level HTTP or database exceptions into upper domain or application layers.

## 4. Concurrency & Resource Control
- **Explicit Async Safeguards:** When using `asyncio`, never block the event loop with synchronous, I/O-bound, or CPU-heavy calls. Wrap blocking code explicitly in `asyncio.to_thread()` or run it within a designated `ProcessPoolExecutor`.
- **Timeouts by Default:** Every network call, database query, agent tool execution, and IPC call must have a strict, explicit timeout boundary configured. Never allow an operation to await indefinitely.

## 5. Absolute Banned Anti-Patterns
- **The Global State Trap:** Use of the `global` keyword, mutable module-level variables, or un-synchronized singletons that break thread/coroutine safety in concurrent agent executions is completely forbidden.
- **Magic Numbers & Strings:** Inline literal primitives used for configurations, timeouts, or states are banned. Use explicit `Enum` classes or structured configuration objects.
