---
name: gitignore-skeleton
description: Standard .gitignore structure (AI Tools section first, Secrets, project-type, Editors, OS). Creates from scratch or updates existing files.
mode: agent
category: git
shared: true
---

# .gitignore Skeleton

Unified skill for writing project `.gitignore` files following a consistent pattern. Handles both new projects and updating existing files that deviate from the standard.

## Context Detection

Before starting, determine the context:

1. **Check for `.gitignore`** in the current directory (or the target directory):
   ```bash
   find . -name ".gitignore" -maxdepth 1
   ```

2. **If `.gitignore` exists** → this is an existing file that needs updating. Read `references/migrate.md`.

3. **If `.gitignore` does not exist** → create from scratch. Read `references/create.md`.

4. **Always read `references/create.md`** — it contains the canonical section order and content for both flows.

5. **Detect project type**: look for `go.mod` (Go), `pyproject.toml` / `.venv` (Python), `package.json` (Node) — this determines which project-type section to include.

## Reference Files

| File | When to read |
|------|-------------|
| `references/create.md` | Always — canonical section order and complete examples |
| `references/migrate.md` | Updating an existing `.gitignore` |

## Related Skills

- **go-skeleton** — Go project layout that informs the Go section entries
- **python-skeleton** — Python project layout that informs the Python section entries
