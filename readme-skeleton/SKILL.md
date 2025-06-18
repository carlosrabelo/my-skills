---
name: readme-skeleton
description: Standard README structure (section order, Highlights format, Development section) plus bilingual sync (README-PT.md). Creates from scratch or updates existing files.
mode: agent
category: documentation
shared: true
---

# README Skeleton

Unified skill for writing project README files following a consistent pattern. Handles both new projects and updating existing files that deviate from the standard. Always handles both `README.md` and `README-PT.md` together.

## Context Detection

Before starting, determine the context:

1. **Check for `README.md`** in the current directory (or the target directory):
   ```bash
   find . -name "README.md" -maxdepth 1
   ```

2. **If `README.md` exists** → this is an existing file that needs updating. Read `references/migrate.md`.

3. **If `README.md` does not exist** → create from scratch. Read `references/create.md`.

4. **Always read `references/create.md`** — it contains the canonical section order and format rules for both flows.

5. **Always read `references/bilingual.md`** — after writing or updating `README.md`, create or sync `README-PT.md` as a Portuguese translation.

## Reference Files

| File | When to read |
|------|-------------|
| `references/create.md` | Always — canonical structure and section rules |
| `references/migrate.md` | Updating an existing README.md |
| `references/bilingual.md` | Always — translation rules for README-PT.md |

## Related Skills

- **go-skeleton** — Go project layout that Project Layout section should reflect
- **python-skeleton** — Python project layout that Project Layout section should reflect
- **makefile-skeleton** — The make targets shown in the Development section
- **github-repo-editor** — Update GitHub repository description and topics after the README is ready
