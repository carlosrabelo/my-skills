---
name: gitignore-skeleton
description: Standard .gitignore structure (Secrets, project-type, Editors, OS). Creates from scratch or updates existing files.
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
- [ ] `# SECRETS` section present with `.env` and credentials (Scenario 1)
- [ ] Project-type section complete and accurate (Scenario 2: GO, PYTHON, NODE, CPP, or ANSIBLE)
- [ ] Sections in priority order: SECRETS → Project → EDITORS → OS → EXCEPTIONS (Scenario 3)
- [ ] `# EDITORS` and `# OS` sections present (Scenario 4)
- [ ] All section headers use `====` banner format and ALL CAPS titles (Scenario 5)
- [ ] `# EXCEPTIONS` section at the bottom if `.gitkeep` files need to be preserved (Scenario 2/Ansible)

### Migration Scenarios

#### Scenario 1: Missing Secrets Section

**When**: The `.gitignore` does not ignore `.env` or credential files.

**Add at the very top**:

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

#### Scenario 2: Missing or Incomplete Project-Type Section

**When**: The `.gitignore` is missing entries for the detected stack, or has only partial coverage.

**Go — missing or incomplete**

```gitignore
# ============================================================================
# GO
# ============================================================================

# Directories
bin/*
!bin/.gitkeep

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

**C++ — missing or incomplete**

Detect by presence of `CMakeLists.txt`, multiple `*.cpp` / `*.hpp`, `compile_commands.json`, or a Makefile-driven C++ toolchain (and the project is not ESP32-only — then see **ESP32** / **cpp-skeleton** as appropriate).

```gitignore
# ============================================================================
# CPP
# ============================================================================

# Directories
bin/*
!bin/.gitkeep
obj/

# Files
*.a
*.d
*.o
*.out
*.so
compile_commands.json
```

Place `!bin/.gitkeep` immediately after `bin/*`. Add `bin/.gitkeep` in the repo when using this pattern.

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

#### Scenario 3: Sections Out of Priority Order

**When**: The existing sections exist but are not in the correct order.

**Correct order** (top = highest priority):

```
# SECRETS
# [GO | PYTHON | NODE | CPP | ANSIBLE | other project-type]
# EDITORS
# OS
# EXCEPTIONS   ← only if needed; always last
```

Steps:
1. Extract each section as a block (from the `====` banner to the blank line before the next).
2. Reorder the blocks to match the priority sequence above.
3. Keep a single blank line between sections.

---

#### Scenario 4: Missing Editors or OS Sections

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

#### Scenario 5: Missing Banner Format or Lowercase Section Titles

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

- **Preserve all existing entries** — migration adds and reorders; it does not remove entries unless they are duplicates or clearly wrong.
- **One blank line between sections** — consistent formatting makes the file scannable.
- **Do not add project-type sections that don't apply** — a Go project does not need `node_modules/`.
- **Section titles are ALL CAPS** — `# SECRETS`, `# GO`, `# PYTHON`, `# NODE`, `# CPP`, `# ANSIBLE`, `# EDITORS`, `# OS`, `# EXCEPTIONS`.
- **Every section uses the `====` banner format** — no plain `# Name` headers.

---

## Creating from Scratch

Every project needs a `.gitignore`. The order of sections reflects priority — the most critical entries come first so they are never accidentally removed or overlooked during edits.

**Key principle**: SECRETS is always present and always first. Credentials and local state must never be committed.

Write the file using the sections defined in ## Canonical .gitignore Format below, selecting the project-type section that matches the detected stack.

---

## Canonical .gitignore Format

Sections in priority order (top = highest priority):

```
# SECRETS           ← always first, always present
# [Project type]    ← GO | PYTHON | NODE | CPP | ANSIBLE | other
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

### Section 1 — SECRETS (mandatory, always first)

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

### Section 2 — Project Type (pick one or combine)

**GO**

```gitignore
# ============================================================================
# GO
# ============================================================================

# Directories
bin/*
!bin/.gitkeep

# Files
project-name
coverage.out
coverage.html
```

Replace `project-name` with the actual binary name. This catches misplaced `go build` calls that land a binary at the project root instead of `bin/` (see `go-skeleton`). Use `bin/*` plus `!bin/.gitkeep` so the `bin/` tree stays in version control (empty dir via placeholder) while all built binaries are ignored — place the negation immediately after `bin/*`.

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

**CPP**

```gitignore
# ============================================================================
# CPP
# ============================================================================

# Directories
bin/*
!bin/.gitkeep
obj/

# Files
*.a
*.d
*.o
*.out
*.so
compile_commands.json
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

### Section 3 — EDITORS (mandatory)

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

### Section 4 — OS (mandatory)

```gitignore
# ============================================================================
# OS
# ============================================================================

# Files
.DS_Store
Thumbs.db
```

### Section 5 — EXCEPTIONS (optional, always last)

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
- **Go** normally uses `bin/*` then `!bin/.gitkeep` inside the **GO** section (not here) so the negation immediately follows the ignore rule.

### Complete Examples

**Go Project**

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

# ============================================================================
# GO
# ============================================================================

# Directories
bin/*
!bin/.gitkeep

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

- **Plain `# Name` section headers** — always use the `====` banner format with ALL CAPS title.

- **No Directories/Files subsections** — entries without grouping are harder to scan; always split into `# Directories` and `# Files`.

- **Committing `.env`** — always ignore it. Provide `.env.example` instead.

- **Ignoring `node_modules/` in a Go project** — only include the section that matches the project type.

- **Using `*` globs broadly** — prefer explicit entries over `*` wildcards that may accidentally block legitimate files. **Exception:** `bin/*` with `!bin/.gitkeep` is the standard Go pattern for build output while tracking an empty `bin/` directory.

- **EXCEPTIONS section not last** — negations placed before the ignore rules they override have no effect.

---

## Monorepo

In a monorepo the SECRETS, EDITORS and OS sections live **only in the root `.gitignore`**. Each component has its own `.gitignore` containing only its project-type section. This avoids duplicating boilerplate in every component.

```
monorepo/
├── .gitignore          ← SECRETS + EDITORS + OS
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
bin/*
!bin/.gitkeep

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

- [ ] Root `.gitignore` has SECRETS, EDITORS, OS — no project-type sections
- [ ] Each component has its own `.gitignore` with only its project-type section
- [ ] No duplicate SECRETS blocks inside components
- [ ] All section headers use `====` banner format and ALL CAPS titles

---

## Related Skills

- **go-skeleton** — Go project layout that informs the Go section entries
- **python-skeleton** — Python project layout that informs the Python section entries
- **monorepo-skeleton** — Monorepo layout this skill supports (root orchestrator + self-contained components)
