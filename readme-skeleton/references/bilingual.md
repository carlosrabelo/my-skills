# Bilingual README

Translate `README.md` into `README-PT.md` and keep both files in sync.

## Overview

`README.md` is the English source of truth. `README-PT.md` is a complete Portuguese translation. Both files must have identical section structure and identical section order at all times.

Use when:
- Creating `README-PT.md` for the first time
- `README.md` was updated and `README-PT.md` needs to catch up
- The two files have drifted and need to be reconciled

---

## Section Heading Translation Reference

| English | Portuguese |
|---|---|
| Highlights | Destaques |
| Table of Contents | Sumário |
| Overview | Visão Geral |
| Prerequisites | Pré-requisitos |
| Installation | Instalação |
| Quick Start | Início Rápido |
| Usage | Uso |
| Configuration | Configuração |
| Project Layout | Estrutura do Projeto |
| Development | Desenvolvimento |
| Contributing | Contribuição |
| License | Licença |

---

## What to Translate

**Translate:**
- Section headings (use the reference table above)
- All prose: descriptions, explanations, instructions, bullet points

**Do NOT translate:**
- Code inside fenced code blocks
- Command names: `make build`, `git clone`, etc.
- File paths: `~/.local/bin`, `cmd/project-name/`, etc.
- Technical terms: Git, CLI, API, Go, Python, JSON, YAML, etc.
- Variable and function names
- URLs and links (translate the anchor text only if it is prose)

**Optional:**
- Inline comments inside code blocks (the `# comment` part) — translate when the comment explains a step to the user

---

## Creating README-PT.md from Scratch

1. Copy `README.md` to `README-PT.md`.
2. Translate all section headings using the reference table.
3. Translate all prose.
4. Leave code blocks, commands, paths, and technical terms untouched.
5. Verify structure parity:
   ```bash
   diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)
   ```
   The diff must be empty.

---

## Syncing After README.md Changes

1. Identify what changed in `README.md` (new sections, updated content, removed sections).
2. Apply each change to `README-PT.md`:
   - New section → translate and insert in the same position
   - Updated content → translate the updated prose and replace the corresponding PT section
   - Removed section → remove from PT as well
   - Reordered sections → reorder PT to match
3. Verify structure parity after changes:
   ```bash
   diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)
   ```

---

## Translation Quality

- Write clear, professional Brazilian Portuguese
- Match the tone and register of the English version
- Keep sentences direct — Portuguese prose that is shorter than the English is usually better
- Do not add information that is not in the English version
- Do not omit information that is in the English version

---

## Rules

- Never commit one file without the other — always `git add README.md README-PT.md` together
- The English file is always the source of truth — if in doubt, follow the English
- Section order must be identical in both files — the `diff` check is the gate
