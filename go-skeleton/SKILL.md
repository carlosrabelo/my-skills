---
name: go-skeleton
description: Standard Go project structure (go.mod at root, cmd/, internal/, make/ scripts). Creates from scratch or reorganizes existing projects.
mode: agent
category: go
shared: true
---

# Go Skeleton

Unified skill for organizing Go projects following modern Go conventions. Handles both new projects and reorganization of existing ones.

## Context Detection

Before starting, determine the context:

1. **Check for `go.mod`** in the current directory (or the target directory):
   ```bash
   find . -name "go.mod" -not -path "./vendor/*" -maxdepth 3
   ```

2. **If `go.mod` exists** → this is an existing project that needs reorganization. Read `references/migrate.md`.

3. **If `go.mod` does not exist** (or the directory is empty/new) → this is a new project being created from scratch. Read `references/create.md`.

4. **Always read `references/layout.md`** — it is the canonical target structure for both flows.

5. **Read `references/patterns.md`** when the task involves testing patterns, error handling, anti-patterns, or code organization decisions.

## Reference Files

| File | When to read |
|------|-------------|
| `references/layout.md` | Always — canonical structure |
| `references/create.md` | Creating a new project from scratch |
| `references/migrate.md` | Reorganizing an existing Go project |
| `references/patterns.md` | Testing, error handling, anti-patterns, code comments |

## Monorepo Usage

This skill applies to whichever directory contains `go.mod` — that is the Go project root, regardless of where the git root is.

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

## Chaining

When creating a complete Go project from scratch, the full workflow involves these skills in order:

1. **`gitignore-skeleton`** — `.gitignore` with Go patterns (binaries, vendor, caches)
2. **`readme-skeleton`** — `README.md` with the standard structure (handles `README-PT.md` automatically)

## Related Skills

- **makefile-skeleton** — Generic Makefile structure conventions (opening lines, .PHONY, help pattern)
- **readme-skeleton** — Standard README content, section order, and bilingual conventions
- **gitignore-skeleton** — For bringing `.gitignore` up to the standard after reorganization
- **monorepo-skeleton** — For organizing multi-language monorepos with Go as one component
