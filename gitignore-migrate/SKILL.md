---
name: gitignore-migrate
description: Bring an existing .gitignore up to the standard structure. Covers adding the mandatory AI Tools section, inserting missing secrets entries, reordering sections by priority, and adding missing project-type entries for Go, Python, and Node.
mode: manual
category: git
shared: true
---

# .gitignore Migration

Step-by-step guide for bringing an existing `.gitignore` up to the standard defined in `gitignore-create`.

## Overview

Use this skill when a project has an existing `.gitignore` that is missing critical sections or has entries out of priority order. Invoke manually with `/gitignore-migrate`.

**Target state**: `.gitignore` with:
- AI Tools section always present and always first
- Secrets section present
- Project-type section matching the stack (Go, Python, or Node)
- Editors and OS sections present
- Sections in the correct priority order

---

## Before You Start

1. Read the existing `.gitignore` in full.
2. Detect the project type: look for `go.mod` (Go), `pyproject.toml` / `.venv` (Python), `package.json` (Node).
3. Note which scenarios apply before making any changes — several are often needed together.

---

## Migration Scenarios

### Scenario 1: Missing AI Tools Section

**When**: The `.gitignore` has no `# AI Tools` section.

This is the most common gap. AI tool artifacts are never committed by default but have no standard way to enter the repo unless explicitly blocked.

**Add at the very top** (before everything else):

```gitignore
# AI Tools
CLAUDE.md
CLAUDE.local.md
.claude/
GEMINI.md
.gemini/
.antigravity/
.opencode/
opencode.json
.cursor/
.aider
.aider.conf.yml
```

If the project intentionally commits `CLAUDE.md` or `GEMINI.md` (e.g., a tools or skills repo), add a negation immediately after the block:

```gitignore
!CLAUDE.md
```

---

### Scenario 2: Missing Secrets Section

**When**: The `.gitignore` does not ignore `.env` or credential files.

**Add after the AI Tools section**:

```gitignore
# Secrets
.env
.env.*
!.env.example
*.key
*.pem
credentials.json
```

If `.env.example` should also be ignored (no template needed), remove the `!.env.example` negation.

---

### Scenario 3: Missing or Incomplete Project-Type Section

**When**: The `.gitignore` is missing entries for the detected stack, or has only partial coverage.

#### Go — missing or incomplete

Check for `go.mod` in the project root. Add or complete:

```gitignore
# Go
bin/
project-name
coverage.out
coverage.html
```

Replace `project-name` with the actual binary name. This catches misplaced `go build` calls that land a binary at the project root instead of `bin/` (see `go-project-create`).

#### Python — missing or incomplete

Check for `pyproject.toml` or `.venv/`. Add or complete:

```gitignore
# Python
__pycache__/
*.py[cod]
*.pyo
.venv/
*.egg-info/
dist/
build/
MANIFEST
.pytest_cache/
.coverage
.coverage.*
htmlcov/
.tox/
.mypy_cache/
.pytype/
.ruff_cache/
```

#### Node — missing or incomplete

Check for `package.json`. Add or complete:

```gitignore
# Node
node_modules/
dist/
.next/
out/
*.tsbuildinfo
```

---

### Scenario 4: Sections Out of Priority Order

**When**: The existing sections exist but are not in the correct order.

**Correct order** (top = highest priority):

```
# AI Tools
# Secrets
# [Go | Python | Node | other project-type]
# Editors
# OS
```

Steps:
1. Extract each section as a block (from one `# Heading` comment to the blank line before the next).
2. Reorder the blocks to match the priority sequence above.
3. Keep a single blank line between sections.

---

### Scenario 5: Missing Editors or OS Sections

**When**: The `.gitignore` ignores project files but has no entries for editor artifacts or OS files.

**Add Editors section** (after project-type, before OS):

```gitignore
# Editors
.idea/
.vscode/
*.swp
*.swo
*~
```

**Add OS section** (always last):

```gitignore
# OS
.DS_Store
Thumbs.db
```

---

## Migration Checklist

**Before:**
- [ ] Read the current `.gitignore` in full
- [ ] Detect project type (`go.mod`, `pyproject.toml`, `package.json`)
- [ ] Note which scenarios apply

**After:**
- [ ] `# AI Tools` section present and at the top (Scenario 1)
- [ ] `# Secrets` section present with `.env` and credentials (Scenario 2)
- [ ] Project-type section complete and accurate (Scenario 3)
- [ ] Sections in priority order: AI → Secrets → Project → Editors → OS (Scenario 4)
- [ ] `# Editors` and `# OS` sections present (Scenario 5)

---

## Rules

- **AI Tools section is always first** — never reorder it below any other section.
- **Preserve all existing entries** — migration adds and reorders; it does not remove entries unless they are duplicates or clearly wrong.
- **One blank line between sections** — consistent formatting makes the file scannable.
- **Do not add project-type sections that don't apply** — a Go project does not need `node_modules/`.

---

## Related Skills

- `gitignore-create` — the target state this skill migrates toward
- `go-project-create` — Go project layout that informs the Go section entries
- `python-project-create` — Python project layout that informs the Python section entries
- `go-project-migrate` — invokes this skill as part of its reorganization checklist
- `python-project-migrate` — invokes this skill as part of its reorganization checklist
