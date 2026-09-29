# 001: Use the `src/rolelens/` package layout

**Date:** 2026-09-28

## Context

The repo was created with a plain `uv init`, giving a flat layout (`main.py` at the root, no package). RoleLens will have several entry points that share code: an ingestion command, a FastAPI server, eval scripts, and tests.

## Decision

Use the `src/` layout with a single top-level package: `src/rolelens/`. The project is built with `uv_build` and installed into the venv, so everything imports it as `rolelens.*` (e.g. `from rolelens.db import ...`).

## Alternatives

- **Flat layout (`main.py` + modules at the root):** fine for a single script. With several entry points, Python can import code straight from the working directory, so a missing file or packaging mistake can pass locally and fail elsewhere.
- **Subpackages directly under `src/` (`src/db/`, `src/agent/`):** `src/` is a container, not a package, so `db`, `agent` and `api` would become top-level import names. They're generic enough to clash with installed libraries, and they don't show the code is ours. Making `src` itself the package (`from src.db import ...`) is awkward and defeats the point of the layout.

## Consequences

- Every entry point imports one installed package the same way, whatever the working directory.
- Packaging mistakes show up early, because only installed code is importable.
- One more folder level, and the project must be installed (`uv sync` does this) before its code can run.
