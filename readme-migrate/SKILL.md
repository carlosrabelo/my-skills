---
name: readme-migrate
description: Bring an existing README.md up to the standard structure. Covers reordering sections, converting Features to Highlights, adding missing Development section, and fixing Project Layout (removing src/). For the Portuguese translation, use readme-bilingual.
mode: manual
category: documentation
shared: true
---

# README Migration

Step-by-step guide for bringing existing README files up to the standard defined in `readme-create`.

## Overview

Use this skill when a project has an existing README.md that deviates from the standard — wrong section order, misnamed sections, missing `## Development`, or `src/` in the layout. For the Portuguese translation, use `readme-bilingual`.

**Target state**: `README.md` with:
- Identical section structure in the canonical order
- `## Highlights` (not `## Features`) with plain bullets
- `## Project Layout` (not `## Project Structure`)
- `## Development` always present with make targets
- No `src/` in the Project Layout block

---

## Before You Start

1. List all current sections: `grep "^## " README.md`
2. Compare against the standard order from `readme-create`.
3. Note which scenarios apply before making any changes — several scenarios are often needed together.

---

## Migration Scenarios

### Scenario 1: Reorder Sections to Match Standard

**When**: Sections exist but are in the wrong order (e.g., Development before Usage, Configuration before Installation).

**Standard order** (required sections in bold):
```
# Title + description
[Badges]
## Highlights          ← required
## Table of Contents   ← optional
## Overview            ← optional
## Prerequisites       ← required if non-obvious
## Installation        ← required
## Quick Start         ← optional
## Usage               ← required
## Configuration       ← optional
## Project Layout      ← optional
## Development         ← required
## Contributing        ← optional
## License             ← required
```

Steps:
1. Extract each section as a block (from one `##` heading to the next).
2. Reorder the blocks to match the standard sequence.

---

### Scenario 2: Convert Features/Resources to Highlights

**When**: The README uses `## Features` / `## Recursos` instead of `## Highlights` / `## Destaques`, or uses a bold-label bullet style.

**Before (section name):**
```markdown
## Features

- **Encryption**: Encrypt private keys with BIP38
- **Decryption**: Decrypt encrypted keys
- **Key generation**: Generate new keypairs
```

**After:**
```markdown
## Highlights

- Encrypt private keys using spec-compliant BIP38 routines
- Decrypt BIP38-encrypted keys with passphrase verification
- Generate fresh keypairs for any Bitcoin network
```

Steps:
1. Rename `## Features` → `## Highlights`.
2. Reformat bold-label bullets to plain bullets:
   - Remove the `**Label**: ` prefix
   - Merge label and description into one plain sentence
   - Remove trailing periods
3. Aim for 6–10 bullets; combine overly granular items if needed.

---

### Scenario 3: Add Missing Development Section

**When**: The README has no `## Development` section at all.

Add the section before `## Contributing` (if present) or before `## License`:

**For a Go project:**
```markdown
## Development

```bash
make build      # Compile binary to bin/project-name
make test       # Run all tests
make quality    # Format, vet, and lint
make install    # Install to ~/.local/bin
```
```

**For a Python project:**
```markdown
## Development

```bash
make setup      # Create .venv and install dependencies (first time only)
make test       # Run all tests
make quality    # Format, lint, and type-check
make install    # Install entry points
```
```

---

### Scenario 4: Fix or Add Project Layout

**When**: The section is named `## Project Structure`, or it contains `src/` entries that conflict with the flat root layout.

**Rename the section:**
- `README.md`: `## Project Structure` → `## Project Layout`
- `README-PT.md`: `## Estrutura do Projeto` stays the same (correct translation for `## Project Layout`)

**Remove `src/` entries:**

Before:
```
project/
├── src/
│   ├── cmd/
│   └── internal/
├── bin/
└── make/
```

After:
```
cmd/project-name/    # Go entry point
internal/            # Internal packages
bin/                 # Compiled binaries
make/                 # Build and install scripts
```

Steps:
1. Rename the section heading.
2. Remove any `src/` directory from the tree — move its children up one level.
3. Add `make/` entry if missing.

---

## Migration Checklist

**Before:**
- [ ] `grep "^## " README.md` — note current sections and order
- [ ] Note which scenarios apply

**After:**
- [ ] Section order matches standard (Scenario 1 check)
- [ ] `## Highlights` present with plain bullets (Scenario 2 check)
- [ ] `## Development` present with make targets (Scenario 3 check)
- [ ] `## Project Layout` used (not `## Project Structure`), no `src/` entries (Scenario 4 check)

---

---

## Related Skills

- `readme-create` — the target state this skill migrates toward
- `readme-bilingual` — create or update the Portuguese translation after migrating README.md
- `go-project-create` — the directory layout that Project Layout should reflect for Go projects
- `python-project-create` — same, for Python projects
- `go-project-migrate` — invokes this skill as part of its Chaining sequence
- `python-project-migrate` — invokes this skill as part of its Chaining sequence
