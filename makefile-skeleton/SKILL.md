---
name: makefile-skeleton
description: Standard Makefile structure (opening lines, .PHONY, self-doc help, delegate-to-make/ pattern). Creates from scratch or updates existing files.
mode: agent
category: tooling
shared: true
---

# Makefile Skeleton

Unified skill for writing project Makefiles following a consistent pattern. Handles both new projects and updating existing files that deviate from the standard.

## Context Detection

Before starting, determine the context:

1. **Check for `Makefile`** in the current directory (or the target directory):
   ```bash
   find . -name "Makefile" -maxdepth 1
   ```

2. **If `Makefile` exists** → this is an existing file that needs updating. Read `references/migrate.md`.

3. **If `Makefile` does not exist** → create from scratch. Read `references/create.md`.

4. **Always read `references/create.md`** — it contains the canonical structure and format rules for both flows.

## Reference Files

| File | When to read |
|------|-------------|
| `references/create.md` | Always — canonical structure, targets, and examples |
| `references/migrate.md` | Updating an existing Makefile |

## Monorepo Usage

In a monorepo, the root Makefile is an orchestrator. It delegates to component scripts — it does not contain build logic itself. Component Makefiles each follow the same standard structure.

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

## Related Skills

- **go-skeleton** — Complete Go Makefile with `BINARY_NAME`, `LDFLAGS`, `VERSION`, and `make/` scripts
- **python-skeleton** — Complete Python Makefile with `setup`, `typecheck`, `.venv`, and `make/setup.sh`
- **esp32-skeleton** — Complete ESP32 Makefile with `flash`, `monitor`, `install-pio`, and PlatformIO scripts
- **monorepo-skeleton** — Root orchestrator Makefile pattern for multi-component repos
- **project-scaffold** — Auto-detects project type and applies the appropriate skeleton
