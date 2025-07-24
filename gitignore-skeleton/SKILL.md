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

2. **If `.gitignore` exists** → this is an existing file that needs updating. Follow ## Migrating an Existing .gitignore below.

3. **If `.gitignore` does not exist** → create from scratch. Follow ## Creating from Scratch below.

4. **Detect project type**: look for `go.mod` (Go), `pyproject.toml` / `.venv` (Python), `package.json` (Node) — this determines which project-type section to include.

---

## Migrating an Existing .gitignore

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the .gitignore looks "mostly correct".**

### Mandatory Checklist

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

### Migration Scenarios

#### Scenario 1: Missing AI Tools Section

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

#### Scenario 2: Missing Secrets Section

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

#### Scenario 3: Missing or Incomplete Project-Type Section

**When**: The `.gitignore` is missing entries for the detected stack, or has only partial coverage.

**Go — missing or incomplete**

Check for `go.mod` in the project root. Add or complete:

```gitignore
# Go
bin/
project-name
coverage.out
coverage.html
```

Replace `project-name` with the actual binary name. This catches misplaced `go build` calls that land a binary at the project root instead of `bin/` (see `go-skeleton`).

**Python — missing or incomplete**

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

**Node — missing or incomplete**

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

#### Scenario 4: Sections Out of Priority Order

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

#### Scenario 5: Missing Editors or OS Sections

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

### Migration Rules

- **AI Tools section is always first** — never reorder it below any other section.
- **Preserve all existing entries** — migration adds and reorders; it does not remove entries unless they are duplicates or clearly wrong.
- **One blank line between sections** — consistent formatting makes the file scannable.
- **Do not add project-type sections that don't apply** — a Go project does not need `node_modules/`.

---

## Creating from Scratch

Every project needs a `.gitignore`. The order of sections reflects priority — the most critical entries come first so they are never accidentally removed or overlooked during edits.

**Key principle**: The AI Tools section is always present and always first, regardless of project type. AI config files, credentials, and local state must never be committed.

Write the file using the sections defined in ## Canonical .gitignore Format below, selecting the project-type section that matches the detected stack.

---

## Canonical .gitignore Format

Sections in priority order (top = highest priority):

```
# AI Tools          ← always first, always present
# Secrets           ← always present
# [Project type]    ← Go | Python | Node | other
# Editors           ← always present
# OS                ← always present
```

### Section 1 — AI Tools (mandatory, always first)

Blocks config files, local state, and instruction files generated by or used exclusively by AI coding assistants. These must not be committed to any repository.

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

**Exception**: If the project intentionally ships `CLAUDE.md` or `GEMINI.md` as checked-in instructions (e.g., a skills/tools repo), add a negation after the block:

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
!CLAUDE.md        ← re-allows only the project instruction file
```

### Section 2 — Secrets (mandatory)

```gitignore
# Secrets
.env
.env.*
!.env.example
*.key
*.pem
credentials.json
```

The `!.env.example` negation keeps the example file in version control so contributors can bootstrap their environment.

### Section 3 — Project Type (pick one or combine)

**Go**

```gitignore
# Go
bin/
coverage.out
coverage.html
```

Also add the project binary name without path to catch misplaced `go build` calls (see anti-pattern in `go-skeleton`):

```gitignore
# Go
bin/
project-name
coverage.out
coverage.html
```

**Python**

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

**Node / JavaScript**

```gitignore
# Node
node_modules/
dist/
.next/
out/
*.tsbuildinfo
```

### Section 4 — Editors (mandatory)

```gitignore
# Editors
.idea/
.vscode/
*.swp
*.swo
*~
```

### Section 5 — OS (mandatory)

```gitignore
# OS
.DS_Store
Thumbs.db
```

### Complete Examples

**Go Project**

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

# Secrets
.env
.env.*
!.env.example
*.key
*.pem
credentials.json

# Go
bin/
project-name
coverage.out
coverage.html

# Editors
.idea/
.vscode/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
```

**Python Project**

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

# Secrets
.env
.env.*
!.env.example
*.key
*.pem
credentials.json

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

# Editors
.idea/
.vscode/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
```

---

## Anti-Patterns

- **Missing AI Tools section** — AI config files land in version control silently. Always include it first.

- **AI Tools section not first** — prioritize it at the top so it survives future edits.

- **Committing `.env`** — always ignore it. Provide `.env.example` instead.

- **Ignoring `node_modules/` in a Go project** — only include the section that matches the project type.

- **Using `*` globs broadly** — prefer explicit entries over `*` wildcards that may accidentally block legitimate files.

---

## Related Skills

- **go-skeleton** — Go project layout that informs the Go section entries
- **python-skeleton** — Python project layout that informs the Python section entries
