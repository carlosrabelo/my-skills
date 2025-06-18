---
name: monorepo-skeleton
description: Standard multi-language monorepo structure (component-level roots, root orchestrator Makefile, consistent naming). Creates from scratch or reorganizes existing repos.
mode: agent
category: project
shared: true
---

# Monorepo Skeleton

Unified skill for organizing multi-language monorepos following a consistent pattern. Handles both new monorepos and reorganization of existing repos.

## Context Detection

Before starting, determine the context:

1. **Check for language roots at the git root**:
   ```bash
   ls go.mod pyproject.toml package.json 2>/dev/null
   ```

2. **If language files exist at the git root** → this repo needs reorganization into the monorepo layout. Read `references/migrate.md`.

3. **If no language files exist at the git root** (or the directory is empty/new) → create the monorepo structure from scratch. Read `references/create.md`.

4. **Always read `references/create.md`** — it contains the canonical target structure for both flows.

## Reference Files

| File | When to read |
|------|-------------|
| `references/create.md` | Always — canonical structure and conventions |
| `references/migrate.md` | Reorganizing an existing repo into monorepo layout |

## Related Skills

- **go-skeleton** — Internal structure of Go components (create from scratch or migrate existing)
- **python-skeleton** — Internal structure of Python components (create from scratch or migrate existing)
- **makefile-skeleton** — Standard Makefile structure used in both root orchestrator and component Makefiles
