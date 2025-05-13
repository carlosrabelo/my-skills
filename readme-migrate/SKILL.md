---
name: readme-migrate
description: Reorganize existing README files to match the standard structure. Covers reordering sections, converting Features to Highlights, adding missing Development section, fixing Project Layout (removing src/), and creating README-PT.md when absent.
mode: manual
category: documentation
shared: true
---

# README Migration

Step-by-step guide for bringing existing README files up to the standard defined in `readme-create`.

## Overview

Use this skill when a project has an existing README (or only one language version) that deviates from the standard — wrong section order, misnamed sections, missing `## Development`, `src/` in the layout, or no Portuguese translation.

**Target state**: both `README.md` and `README-PT.md` exist with:
- Identical section structure in the canonical order
- `## Highlights` (not `## Features`) with plain bullets
- `## Project Layout` (not `## Project Structure`)
- `## Development` always present with make targets
- No `src/` in the Project Layout block

---

## Before You Start

1. List all current sections: `grep "^## " README.md`
2. Compare against the standard order from `readme-create`.
3. Check whether `README-PT.md` exists. If not, Scenario 5 applies.
4. If both files exist, check whether their sections match: `grep "^## " README.md` vs `grep "^## " README-PT.md`
5. Note which scenarios apply before making any changes — several scenarios are often needed together.

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
3. Apply the identical reordering to `README-PT.md`.
4. Verify both files have the same section order: `diff <(grep "^## " README.md) <(grep "^## " README-PT.md)`

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
1. Rename `## Features` → `## Highlights` in `README.md`.
2. Rename `## Recursos` → `## Destaques` in `README-PT.md`.
3. Reformat bold-label bullets to plain bullets:
   - Remove the `**Label**: ` prefix
   - Merge label and description into one plain sentence
   - Remove trailing periods
4. Aim for 6–10 bullets; combine overly granular items if needed.

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

Apply the equivalent section in Portuguese to `README-PT.md`:
```markdown
## Desenvolvimento

```bash
make build      # Compila o binário em bin/project-name
make test       # Executa todos os testes
make quality    # Formata, verifica e executa o linter
make install    # Instala em ~/.local/bin
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
└── run/
```

After:
```
cmd/project-name/    # Go entry point
internal/            # Internal packages
bin/                 # Compiled binaries
run/                 # Build and install scripts
```

Steps:
1. Rename the section heading.
2. Remove any `src/` directory from the tree — move its children up one level.
3. Add `run/` entry if missing.
4. Apply identical changes to `README-PT.md` (the tree content is not translated, but inline comments in Portuguese are fine).

---

### Scenario 5: Create README-PT.md from Scratch

**When**: Only `README.md` exists — no Portuguese version.

Steps:
1. Copy `README.md` to `README-PT.md`.
2. Translate all section headings using the translation reference in `readme-create`:
   - `## Highlights` → `## Destaques`
   - `## Prerequisites` → `## Pré-requisitos`
   - `## Installation` → `## Instalação`
   - `## Quick Start` → `## Início Rápido`
   - `## Usage` → `## Uso`
   - `## Configuration` → `## Configuração`
   - `## Project Layout` → `## Estrutura do Projeto`
   - `## Development` → `## Desenvolvimento`
   - `## Contributing` → `## Contribuição`
   - `## License` → `## Licença`
3. Translate all prose (descriptions, instructions, bullet points).
4. Do NOT translate: code blocks, command names, file paths, variable names, technical terms (Git, CLI, API, etc.).
5. Optional: translate inline comments inside code blocks (the `# comment` parts).
6. Verify structure parity: `diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)` — should show no differences.

---

### Scenario 6: Sync Diverged Files

**When**: Both `README.md` and `README-PT.md` exist but have drifted — different section order, a section present in one but not the other, or content significantly out of date in the Portuguese version.

Steps:
1. Identify the differences: `diff <(grep "^## " README.md) <(grep "^## " README-PT.md)`
2. For sections missing from PT: translate and add them in the correct position.
3. For sections missing from EN: check if the PT section is a valid addition (translate back and add) or a mistake (remove from PT).
4. For sections in the wrong order in PT: reorder to match EN.
5. For outdated content in PT: read the EN section, translate the updated content, replace the PT section.
6. After reconciling: verify both files have identical section structure.

---

## Migration Checklist

**Before:**
- [ ] `grep "^## " README.md` — note current sections and order
- [ ] Check whether `README-PT.md` exists
- [ ] If both exist: `diff <(grep "^## " README.md) <(grep "^## " README-PT.md)`
- [ ] Note which scenarios apply

**During:**
- [ ] Apply each scenario to README.md first, then mirror in README-PT.md
- [ ] Never edit only one file — always keep both in sync

**After:**
- [ ] Section order matches standard (Scenario 1 check)
- [ ] `## Highlights` / `## Destaques` present with plain bullets (Scenario 2 check)
- [ ] `## Development` / `## Desenvolvimento` present with make targets (Scenario 3 check)
- [ ] `## Project Layout` used (not `## Project Structure`), no `src/` entries (Scenario 4 check)
- [ ] Both files exist (Scenario 5 check)
- [ ] `diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)` shows no differences
- [ ] Commit both files together: `git add README.md README-PT.md`

---

## Rules

- Always update both files in the same commit — never commit one without the other
- Never translate code, command names, file paths, or technical terms
- Section order must be identical in both files — the diff check is the source of truth
- Do not add or remove sections without applying the same change to both files
- When in doubt about a translation, keep the English term rather than using an incorrect Portuguese one

---

## Encadeamento

Após concluir a migração do README:

1. **Sempre** invoque a skill `readme-bilingual-sync` para verificar e sincronizar `README-PT.md`
2. Após o `readme-bilingual-sync` concluir, invoque a skill `git-commit-suggest` para preparar o commit com as mudanças feitas

---

## Related Skills

- `readme-create` — the target state this skill migrates toward; contains the section translation reference table
- `readme-bilingual-sync` — sync workflow for ongoing maintenance after migration
- `go-project-create` — the directory layout that Project Layout should reflect for Go projects
- `python-project-create` — same, for Python projects
