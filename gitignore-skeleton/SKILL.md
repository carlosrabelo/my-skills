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

4. **Detect project type**: look for `go.mod` (Go), `pyproject.toml` / `.venv` (Python), `package.json` (Node), `site.yml` / `playbook*.yml` / `ansible.cfg` / `roles/` (Ansible) — this determines which project-type section to include.

5. **Detect monorepo**: if no language files exist at the git root but subdirectories contain them (e.g. `api-server/go.mod`, `pipeline/pyproject.toml`), this is a monorepo. Follow ## Monorepo below instead of the single-project flow.

---

## Migrating an Existing .gitignore

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the .gitignore looks "mostly correct".**

### Mandatory Checklist

**Before:**
- [ ] Read the current `.gitignore` in full
- [ ] Detect project type (`go.mod`, `pyproject.toml`, `package.json`, `ansible.cfg`, etc.)
- [ ] Note which scenarios apply

**After:**
- [ ] `# AI TOOLS` section present and at the top (Scenario 1)
- [ ] `# SECRETS` section present with `.env` and credentials (Scenario 2)
- [ ] Project-type section complete and accurate (Scenario 3)
- [ ] Sections in priority order: AI TOOLS → SECRETS → Project → EDITORS → OS → EXCEPTIONS (Scenario 4)
- [ ] `# EDITORS` and `# OS` sections present (Scenario 5)
- [ ] All section headers use `====` banner format and ALL CAPS titles (Scenario 6)
- [ ] `# EXCEPTIONS` section at the bottom if `.gitkeep` files need to be preserved (Scenario 3/Ansible)

### Migration Scenarios

#### Scenario 1: Missing AI Tools Section

**When**: The `.gitignore` has no `# AI TOOLS` section.

This is the most common gap. AI tool artifacts are never committed by default but have no standard way to enter the repo unless explicitly blocked.

**Add at the very top** (before everything else):

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/
```

If the project intentionally commits `CLAUDE.md` or `GEMINI.md` (e.g., a tools or skills repo), add a negation immediately after the block:

```gitignore
# Exceptions — preserve these
!CLAUDE.md
```

---

#### Scenario 2: Missing Secrets Section

**When**: The `.gitignore` does not ignore `.env` or credential files.

**Add after the AI TOOLS section**:

```gitignore
# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
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

```gitignore
# ============================================================================
# GO
# ============================================================================

# Directories
bin/

# Files
project-name
coverage.out
coverage.html
```

Replace `project-name` with the actual binary name.

**Python — missing or incomplete**

```gitignore
# ============================================================================
# PYTHON
# ============================================================================

# Directories
__pycache__/
.venv/
*.egg-info/
dist/
build/
.pytest_cache/
htmlcov/
.tox/
.mypy_cache/
.pytype/
.ruff_cache/

# Files
*.py[cod]
*.pyo
MANIFEST
.coverage
.coverage.*
```

**Node — missing or incomplete**

```gitignore
# ============================================================================
# NODE
# ============================================================================

# Directories
node_modules/
dist/
.next/
out/

# Files
*.tsbuildinfo
```

**Ansible — missing or incomplete**

Detect by presence of `site.yml`, `playbook*.yml`, `ansible.cfg`, or `roles/`.

```gitignore
# ============================================================================
# ANSIBLE
# ============================================================================

# Directories
inventory/
group_vars/

# Files
*.retry
*.log
ansible.log
```

If the project has `inventory/` or `group_vars/` directories with `.gitkeep` files, also add an `# EXCEPTIONS` section **at the bottom**:

```gitignore
# ============================================================================
# EXCEPTIONS (preserve these)
# ============================================================================

# Inventory — preserve placeholder
!inventory/.gitkeep

# Variables — preserve placeholder
!group_vars/all/.gitkeep
```

---

#### Scenario 4: Sections Out of Priority Order

**When**: The existing sections exist but are not in the correct order.

**Correct order** (top = highest priority):

```
# AI TOOLS
# SECRETS
# [GO | PYTHON | NODE | ANSIBLE | other project-type]
# EDITORS
# OS
# EXCEPTIONS   ← only if needed; always last
```

Steps:
1. Extract each section as a block (from the `====` banner to the blank line before the next).
2. Reorder the blocks to match the priority sequence above.
3. Keep a single blank line between sections.

---

#### Scenario 5: Missing Editors or OS Sections

**When**: The `.gitignore` ignores project files but has no entries for editor artifacts or OS files.

**Add EDITORS section** (after project-type, before OS):

```gitignore
# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo
```

**Add OS section** (always last, before EXCEPTIONS if any):

```gitignore
# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

---

#### Scenario 6: Missing Banner Format or Lowercase Section Titles

**When**: Section headers use plain `# Name` comments instead of the `====` banner format, or titles are not ALL CAPS.

**Convert every section header** from:
```gitignore
# AI Tools
```

To:
```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================
```

Also convert entries to use `# Directories` / `# Files` subsections within each section.

---

### Migration Rules

- **AI TOOLS section is always first** — never reorder it below any other section.
- **Preserve all existing entries** — migration adds and reorders; it does not remove entries unless they are duplicates or clearly wrong.
- **One blank line between sections** — consistent formatting makes the file scannable.
- **Do not add project-type sections that don't apply** — a Go project does not need `node_modules/`.
- **Section titles are ALL CAPS** — `# AI TOOLS`, `# SECRETS`, `# GO`, `# EDITORS`, `# OS`, `# EXCEPTIONS`.
- **Every section uses the `====` banner format** — no plain `# Name` headers.

---

## Creating from Scratch

Every project needs a `.gitignore`. The order of sections reflects priority — the most critical entries come first so they are never accidentally removed or overlooked during edits.

**Key principle**: The AI TOOLS section is always present and always first, regardless of project type. AI config files, credentials, and local state must never be committed.

Write the file using the sections defined in ## Canonical .gitignore Format below, selecting the project-type section that matches the detected stack.

---

## Canonical .gitignore Format

Sections in priority order (top = highest priority):

```
# AI TOOLS          ← always first, always present
# SECRETS           ← always present
# [Project type]    ← GO | PYTHON | NODE | ANSIBLE | other
# EDITORS           ← always present
# OS                ← always present
# EXCEPTIONS        ← only when needed; always last
```

### Formatting Rules

- Every section has a `====` banner header spanning 76 characters.
- Section titles are ALL CAPS.
- Entries within each section are grouped under `# Directories` and `# Files` subsections (omit a subsection only if it has no entries).
- One blank line between the banner and the first subsection.
- One blank line between the last entry of one section and the banner of the next.

### Section 1 — AI TOOLS (mandatory, always first)

Blocks config files, local state, and instruction files generated by or used exclusively by AI coding assistants. These must not be committed to any repository.

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/
```

**Exception**: If the project intentionally ships `CLAUDE.md` or `GEMINI.md` as checked-in instructions (e.g., a skills/tools repo), add a negation after the block:

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/

# Exceptions — preserve project instruction file
!CLAUDE.md
```

### Section 2 — SECRETS (mandatory)

```gitignore
# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
.env
.env.*
!.env.example
*.key
*.pem
credentials.json
```

The `!.env.example` negation keeps the example file in version control so contributors can bootstrap their environment.

### Section 3 — Project Type (pick one or combine)

**GO**

```gitignore
# ============================================================================
# GO
# ============================================================================

# Directories
bin/

# Files
project-name
coverage.out
coverage.html
```

Replace `project-name` with the actual binary name. This catches misplaced `go build` calls that land a binary at the project root instead of `bin/` (see `go-skeleton`).

**PYTHON**

```gitignore
# ============================================================================
# PYTHON
# ============================================================================

# Directories
__pycache__/
.venv/
*.egg-info/
dist/
build/
.pytest_cache/
htmlcov/
.tox/
.mypy_cache/
.pytype/
.ruff_cache/

# Files
*.py[cod]
*.pyo
MANIFEST
.coverage
.coverage.*
```

**NODE**

```gitignore
# ============================================================================
# NODE
# ============================================================================

# Directories
node_modules/
dist/
.next/
out/

# Files
*.tsbuildinfo
```

**ANSIBLE**

```gitignore
# ============================================================================
# ANSIBLE
# ============================================================================

# Directories
inventory/
group_vars/

# Files
*.retry
*.log
ansible.log
```

Detect Ansible projects by presence of `site.yml`, `playbook*.yml`, `ansible.cfg`, or a `roles/` directory.

### Section 4 — EDITORS (mandatory)

```gitignore
# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo
```

### Section 5 — OS (mandatory)

```gitignore
# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

### Section 6 — EXCEPTIONS (optional, always last)

Use when a section above blocks an entire directory but a placeholder file inside must be preserved (e.g., `.gitkeep` to track an otherwise-empty directory in version control).

```gitignore
# ============================================================================
# EXCEPTIONS (preserve these)
# ============================================================================

# Some-dir — preserve placeholder
!some-dir/.gitkeep
```

**Rules**:
- Only include this section when at least one negation is needed.
- Always place it **last** — negations only work if they appear after the ignore rule they override.
- One `!` line per file to preserve; do not use wildcards in negations.
- Add a comment above each negation explaining what directory it belongs to.

### Complete Examples

**Go Project**

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/

# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
.env
.env.*
!.env.example
*.key
*.pem
credentials.json

# ============================================================================
# GO
# ============================================================================

# Directories
bin/

# Files
project-name
coverage.out
coverage.html

# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo

# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

**Python Project**

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/

# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
.env
.env.*
!.env.example
*.key
*.pem
credentials.json

# ============================================================================
# PYTHON
# ============================================================================

# Directories
__pycache__/
.venv/
*.egg-info/
dist/
build/
.pytest_cache/
htmlcov/
.tox/
.mypy_cache/
.pytype/
.ruff_cache/

# Files
*.py[cod]
*.pyo
MANIFEST
.coverage
.coverage.*

# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo

# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

**Ansible Project**

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/

# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
.env
.env.*
*.key
*.pem
credentials.json
vault.yml
*.vault

# ============================================================================
# ANSIBLE
# ============================================================================

# Directories
inventory/
group_vars/

# Files
*.retry
*.log
ansible.log

# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo

# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db

# ============================================================================
# EXCEPTIONS (preserve these)
# ============================================================================

# Inventory — preserve placeholder
!inventory/.gitkeep

# Variables — preserve placeholder
!group_vars/all/.gitkeep
```

Note: Ansible projects use `vault.yml` and `*.vault` for encrypted secrets — add these to the SECRETS section. The `!.env.example` negation is omitted since Ansible projects don't use `.env` templates.

---

## Anti-Patterns

- **Missing AI TOOLS section** — AI config files land in version control silently. Always include it first.

- **AI TOOLS section not first** — prioritize it at the top so it survives future edits.

- **Plain `# Name` section headers** — always use the `====` banner format with ALL CAPS title.

- **No Directories/Files subsections** — entries without grouping are harder to scan; always split into `# Directories` and `# Files`.

- **Committing `.env`** — always ignore it. Provide `.env.example` instead.

- **Ignoring `node_modules/` in a Go project** — only include the section that matches the project type.

- **Using `*` globs broadly** — prefer explicit entries over `*` wildcards that may accidentally block legitimate files.

- **EXCEPTIONS section not last** — negations placed before the ignore rules they override have no effect.

---

## Monorepo

In a monorepo the AI TOOLS, SECRETS, EDITORS and OS sections live **only in the root `.gitignore`**. Each component has its own `.gitignore` containing only its project-type section. This avoids duplicating boilerplate in every component.

```
monorepo/
├── .gitignore          ← AI TOOLS + SECRETS + EDITORS + OS
├── api-server/
│   └── .gitignore      ← GO only
├── pipeline/
│   └── .gitignore      ← PYTHON only
└── infra/
    └── .gitignore      ← ANSIBLE only (+ EXCEPTIONS if needed)
```

### Root `.gitignore`

Contains all shared sections. No project-type section — those belong in components.

```gitignore
# ============================================================================
# AI TOOLS
# ============================================================================

# Directories
.claude/
.gemini/
.antigravity/
.opencode/
.cursor/
.aider/

# Files
CLAUDE.md
CLAUDE.local.md
GEMINI.md
opencode.json
.aider.conf.yml
AGENTS.md
.nexor/

# ============================================================================
# SECRETS
# ============================================================================

# Directories
# (none - secrets are typically files)

# Files
.env
.env.*
!.env.example
*.key
*.pem
credentials.json

# ============================================================================
# EDITORS
# ============================================================================

# Directories
.vscode/
.idea/

# Files
*.swp
*.swo
*~
*.swo

# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

### Component `.gitignore`

Contains only the section for that component's stack.

**Go component** (`api-server/.gitignore`):

```gitignore
# ============================================================================
# GO
# ============================================================================

# Directories
bin/

# Files
api-server
coverage.out
coverage.html
```

**Python component** (`pipeline/.gitignore`):

```gitignore
# ============================================================================
# PYTHON
# ============================================================================

# Directories
__pycache__/
.venv/
*.egg-info/
dist/
build/
.pytest_cache/
htmlcov/
.tox/
.mypy_cache/
.pytype/
.ruff_cache/

# Files
*.py[cod]
*.pyo
MANIFEST
.coverage
.coverage.*
```

**Ansible component** (`infra/.gitignore`):

```gitignore
# ============================================================================
# ANSIBLE
# ============================================================================

# Directories
inventory/
group_vars/

# Files
*.retry
*.log
ansible.log

# ============================================================================
# EXCEPTIONS (preserve these)
# ============================================================================

# Inventory — preserve placeholder
!inventory/.gitkeep

# Variables — preserve placeholder
!group_vars/all/.gitkeep
```

### Monorepo Migration Checklist

When a monorepo has a root `.gitignore` that doesn't follow this pattern:

- [ ] Root `.gitignore` has AI TOOLS, SECRETS, EDITORS, OS — no project-type sections
- [ ] Each component has its own `.gitignore` with only its project-type section
- [ ] No duplicate AI TOOLS or SECRETS blocks inside components
- [ ] All section headers use `====` banner format and ALL CAPS titles

---

## Related Skills

- **go-skeleton** — Go project layout that informs the Go section entries
- **python-skeleton** — Python project layout that informs the Python section entries
- **monorepo-skeleton** — Monorepo layout this skill supports (root orchestrator + self-contained components)
