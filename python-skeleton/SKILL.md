---
name: python-skeleton
description: Standard Python project structure (pyproject.toml, .venv at root, flat layout, shebags). Creates from scratch or reorganizes existing projects.
mode: agent
category: python
shared: true
---

# Python Skeleton

Unified skill for organizing Python projects following a consistent pattern for maintainability and scalability. Handles both new projects and reorganization of existing ones.

## Context Detection

Before starting, determine the context:

1. **Check for `pyproject.toml`** in the current directory (or the target directory):
   ```bash
   find . -name "pyproject.toml" -not -path "./.venv/*" -maxdepth 3
   ```

2. **If `pyproject.toml` exists** → this is an existing project that needs reorganization. Read `references/migrate.md`.

3. **If `pyproject.toml` does not exist** (or the directory is empty/new) → this is a new project being created from scratch. Read `references/create.md`.

4. **Always read `references/layout.md`** — it is the canonical target structure for both flows.

5. **Read `references/patterns.md`** when the task involves testing patterns, error handling, anti-patterns, or shebags.

## Reference Files

| File | When to read |
|------|-------------|
| `references/layout.md` | Always — canonical structure |
| `references/create.md` | Creating a new project from scratch |
| `references/migrate.md` | Reorganizing an existing Python project |
| `references/patterns.md` | Testing, error handling, anti-patterns, shebags |

## Monorepo Usage

This skill applies to whichever directory contains `pyproject.toml` — that is the Python project root.

- `pyproject.toml` and `.venv` live inside `<component>/`, not at the git root
- The self-relaunching shebag works unchanged: `Path(__file__).resolve().parent` resolves to `<component>/`

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

## Chaining

When creating a complete Python project from scratch, the full workflow involves these skills in order:

1. **`makefile-skeleton`** — create the Makefile with standard targets
2. **`readme-skeleton`** — create `README.md` with the standard structure
3. **`readme-skeleton`** handles `README-PT.md` automatically as part of the README flow

## Related Skills

- **makefile-skeleton** — Standard Makefile structure and targets used in every Python project
- **readme-skeleton** — Standard README content and section order
- **gitignore-skeleton** — For bringing `.gitignore` up to the standard after reorganization
- **monorepo-skeleton** — For organizing multi-language monorepos with Python as one component
