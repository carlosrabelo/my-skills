---
name: monorepo-project-create
description: Guide to organizing multi-language monorepos with component-level roots (go.mod / pyproject.toml per component), root orchestrator Makefile, and consistent naming conventions for Go and Python components.
mode: agent
category: project
shared: true
---

# Monorepo Project Structure

## Overview

The git root is an **orchestrator only**. No language "lives" at the git root. Each component is self-contained with its own root (`go.mod` or `pyproject.toml`), its own Makefile, and its own `make/` scripts.

---

## Root Layout

```
monorepo/
├── Makefile              ← Orchestrator only — delegates to component make/ scripts
├── go-component/         ← Go root (contains go.mod)
│   ├── Makefile
│   ├── go.mod
│   ├── bin/
│   └── make/
├── python-component/     ← Python root (contains pyproject.toml)
│   ├── Makefile
│   ├── pyproject.toml
│   ├── .venv/
│   └── make/
└── shared/               ← Optional: CI configs, shared scripts, docs
```

**Never** at the git root:
- `go.mod` or `go.sum`
- `pyproject.toml` or `.venv`
- `bin/` (each component has its own)

---

## Root Orchestrator Makefile

The root Makefile only delegates — it never contains language-specific build or test logic:

```makefile
MAKEFLAGS += --no-print-directory
.PHONY: build test lint

build:
	go-component/make/build.sh
	python-component/make/setup.sh

test:
	go-component/make/test.sh
	python-component/make/test.sh
```

Having two Makefiles (root + component) is **not** Anti-Pattern (Two Makefiles) in a multi-language monorepo — they serve different purposes. The anti-pattern is two Makefiles for the same language/component.

---

## Component Naming

Use domain-descriptive names:

| Good | Bad |
|---|---|
| `api-server/` | `backend/` |
| `data-pipeline/` | `service1/` |
| `cli-tool/` | `src/` |
| `auth-go/`, `auth-py/` (only when names collide) | `lib/` |

Use language suffixes (`-go`, `-py`) only when two components share the same domain name.

---

## Component Self-Containment Rules

Each component:
- Has its own `Makefile` with language-specific targets
- Has its own `make/` with scripts
- Resolves `ROOT_DIR` relative to itself, not the git root:
  ```bash
  ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"   # resolves to <component>/
  ```
- If a script genuinely needs the git root (e.g. shared output path):
  ```bash
  GIT_ROOT="$(cd "$(dirname "$0")/../.." && pwd)"
  ```

---

## Per-Language Components

- **Go components**: see `go-project-create` — `go.mod` lives at `<component>/`, `bin/` and `make/` are siblings
- **Python components**: see `python-project-create` — `pyproject.toml` and `.venv` live at `<component>/`, self-relaunching shebag works unchanged

---

## Anti-Patterns

| Anti-Pattern | Why It's Wrong |
|---|---|
| `go.mod` at git root | Makes the monorepo a pure Go project; other components become sub-packages |
| `pyproject.toml` or `.venv` at git root | Python tooling takes over the entire repo |
| Root Makefile with language-specific build logic | Mixes orchestration with implementation |
| Shared `bin/` at git root | Binary naming conflicts across components |
| Component named `src/` or `lib/` | No domain meaning; misleads future contributors |

---

## When to Use a Monorepo

**Use a monorepo when**:
- Components share a deploy cycle and are developed together
- Teams are small enough to work across all components
- CI/CD benefits from unified pipeline orchestration

**Use separate repos when**:
- Projects are completely independent with different teams
- Release cycles are entirely decoupled
- Sharing code between them is rare or via published packages

---

## Related Skills

- **go-project-create** — Internal structure of Go components
- **python-project-create** — Internal structure of Python components
- **monorepo-project-migrate** — Migrate an existing repo or single-component project to this layout
- **go-project-migrate** — Migrate an existing Go project to the standard layout
- **python-project-migrate** — Migrate an existing Python project to the standard layout
