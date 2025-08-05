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

2. **If `README.md` exists** → this is an existing file that needs updating. Follow ## Migrating an Existing README below.

3. **If `README.md` does not exist** → create from scratch. Follow ## Creating from Scratch and ## Canonical README Structure below.

4. **In both flows**, after writing or updating `README.md`, create or sync `README-PT.md` following ## Bilingual Sync (README-PT.md) below.

---

## Migrating an Existing README

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the README looks "mostly correct".**

### Mandatory Checklist

**Before:**
- [ ] `grep "^## " README.md` — note current sections and order
- [ ] Note which scenarios apply (multiple scenarios often apply together)

**After:**
- [ ] Section order matches standard (Scenario 1 check)
- [ ] `## Highlights` present with plain bullets (Scenario 2 check)
- [ ] `## Development` present with make targets (Scenario 3 check)
- [ ] `## Project Layout` used (not `## Project Structure`), no `src/` entries (Scenario 4 check)

### Migration Scenarios

#### Scenario 1: Reorder Sections to Match Standard

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

#### Scenario 2: Convert Features/Resources to Highlights

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

#### Scenario 3: Add Missing Development Section

**When**: The README has no `## Development` section at all.

Add the section before `## Contributing` (if present) or before `## License`.

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

#### Scenario 4: Fix or Add Project Layout

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
└── .make/
```

After:
```
cmd/project-name/    # Go entry point
internal/            # Internal packages
bin/                 # Compiled binaries
.make/                 # Build and install scripts
```

Steps:
1. Rename the section heading.
2. Remove any `src/` directory from the tree — move its children up one level.
3. Add `.make/` entry if missing.

---

## Creating from Scratch

When no `README.md` exists, build one using the canonical structure and section rules below. Start with the minimal example and add optional sections only when they add real value.

**Key principle**: Include a section only if it adds value. A shorter README with the right information is better than a padded one with filler sections.

---

## Canonical README Structure

The canonical section sequence, with required/optional annotations:

```
# Title + one-line description        ← required
[Badges]                              ← optional
## Highlights                         ← required
## Table of Contents                  ← optional (long READMEs only, ~200+ rendered lines)
## Overview                           ← optional (when one-liner is not enough)
## Prerequisites                      ← required when non-obvious deps exist
## Installation                       ← required
## Quick Start                        ← optional (merge into Installation when simple)
## Usage                              ← required
## Configuration                      ← optional (only when a config system exists)
## Project Layout                     ← optional (helpful for complex projects)
## Development                        ← required
## Contributing                       ← optional (public repos)
## License                            ← required
```

Do not add sections not in this list unless the project genuinely needs them. Do not include sections to pad the document.

---

## Section Writing Rules

### Title and Description

```markdown
# Project Name

One sentence describing what the tool does and who it is for.
```

The description line goes immediately below the title, with no heading. Write in plain present tense. Do not start with "A tool that..." — just describe it directly.

Good: `Command-line application that encrypts and decrypts Bitcoin private keys using the BIP38 standard.`

Bad: `A tool that can be used to encrypt and decrypt Bitcoin private keys.`

---

### Badges (optional)

Include badges only on public-facing repositories that have CI, a versioned release, or a license worth highlighting. Skip badges on private tools, internal scripts, and monorepo components.

Standard badge trio for Go projects:

```markdown
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/Go-1.25%2B-blue.svg)](https://go.dev/)
[![Release](https://img.shields.io/github/release/username/project-name.svg)](https://github.com/username/project-name/releases)
```

Place badges immediately below the one-line description, before `## Highlights`. Do not add more than three or four badges — badge soup obscures the description.

---

### Highlights

The `## Highlights` section is the first section after the title block. It gives a reader the essential "what does this do" in scannable form.

**Format rules:**
- Plain bullet list — one capability per line
- No trailing period
- No bold labels (`**Label**: description` style)
- 6–10 bullets is the sweet spot; fewer is fine, more than 12 is too many

```markdown
## Highlights

- Encrypt and decrypt WIF private keys using spec-compliant BIP38 routines
- Generate fresh WIF keys for any Bitcoin network with optional passphrase protection
- Display native SegWit bech32 addresses for compressed keys
- Hidden terminal input for passphrases to avoid shell history exposure
- Shell completion for bash, zsh, fish, and PowerShell
```

`## Features` is also acceptable if the project already uses it consistently. `## Highlights` is preferred for new projects.

---

### Table of Contents (optional)

Include only when the README is long enough to need navigation (roughly 200+ rendered lines). Use GitHub anchor links:

```markdown
## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Project Layout](#project-layout)
- [Development](#development)
- [License](#license)
```

Anchor syntax: lowercase section name, spaces replaced by hyphens, special characters removed.

---

### Overview (optional)

Include when the one-line description is not enough to orient a new visitor. Keep it to 2–4 sentences covering: what problem does this solve, why does it exist, what makes it different.

Skip it when the project name + one-liner already tells the full story.

---

### Prerequisites

List only genuine prerequisites — things a user might not have and must install before the tool works. Do not list git, make, or other universal tools.

```markdown
## Prerequisites

- **Go 1.25+** — required to build from source; [download](https://go.dev/dl/)
- **golangci-lint** — required for `make lint`; install with `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
```

Bold the tool name, follow with the minimum version and a one-line install note when it is not obvious.

---

### Installation

Always include the "Build from Source" path — it works for every environment. Optional additional paths: `go install`, pre-built binary download.

````markdown
## Installation

### Build from Source

```bash
git clone https://github.com/username/project-name.git
cd project-name
make build
```

Install to `~/.local/bin` (no root required):

```bash
make install
```

### Using Go Install

```bash
go install github.com/username/project-name/cmd/project-name@latest
```
````

Install path convention: `~/.local/bin` for user installs, `/usr/local/bin` for system installs. For Python projects, `make setup` goes before `make build`.

---

### Quick Start (optional)

The minimum path from nothing to first successful output. Under 10 lines including bash blocks. Merge into Installation when the project is very simple.

````markdown
## Quick Start

```bash
make build
./bin/project-name --help
```
````

---

### Usage

Show real command examples with real flags. Each example should be self-contained — do not require readers to have run a previous example.

````markdown
## Usage

### Encrypt a key

```bash
project-name encrypt --wif L1abc...
# Enter passphrase: ****
# 6PY...encrypted-key...
```

### Decrypt a key

```bash
project-name decrypt --key 6PY...
# Enter passphrase: ****
# L1abc...original-wif...
```
````

Show actual output when it clarifies what the command does. Use subheadings for distinct modes or command groups.

Do not write a Usage section that is only prose with no commands.

---

### Configuration (optional)

Include only when the project has a config file or significant environment variables. Skip for simple CLI tools that only have flags.

Show the actual config file format with comments:

````markdown
## Configuration

Create `~/.config/project-name/config.yaml`:

```yaml
# Log verbosity: debug, info, warn, error
log_level: info

# Network: mainnet, testnet, regtest
network: mainnet
```

All options can also be set via environment variables prefixed with `PROJECTNAME_`:

```bash
export PROJECTNAME_LOG_LEVEL=debug
```
````

---

### Project Layout (optional)

Include when the directory structure is non-obvious or when contributors need orientation. Show only meaningful top-level entries with a one-line comment per entry — do not list every file.

Use the section name `## Project Layout` (not `## Project Structure`):

````markdown
## Project Layout

```
cmd/project-name/    # Go entry point
internal/domain/     # Core business logic
internal/cli/        # Command definitions and user interaction
bin/                 # Compiled binaries (git-ignored)
.make/                 # Build and install scripts
cfg/                 # Default configuration files
```
````

Do not show `src/` — it conflicts with the flat root layout used in all Go and Python projects.

---

### Development

Always include this section. It shows the make targets a contributor needs to know.

**For a Go project:**
````markdown
## Development

```bash
make build      # Compile binary to bin/project-name
make test       # Run all tests
make quality    # Format, vet, and lint
make install    # Install to ~/.local/bin
```
````

**For a Python project:**
````markdown
## Development

```bash
make setup      # Create .venv and install dependencies (first time only)
make test       # Run all tests
make quality    # Format, lint, and type-check
make install    # Install entry points
```
````

---

### Contributing (optional)

Include for public open-source projects. Skip for personal tools and internal scripts. Keep it brief:

```markdown
## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/description`
3. Commit with Conventional Commits: `git commit -m "feat: add X"`
4. Push and open a pull request
```

---

### License

Always the last section. One line referencing the LICENSE file:

```markdown
## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
```

---

## Anti-Patterns

- **"A tool that..." in the description** — describes the category, not the tool. Write what it does.
- **Mismatched section structure between README.md and README-PT.md** — the two files must have identical section order. If one has `## Configuration`, the other must too.
- **Omitting `## Development`** — every contributor needs to know how to build and test. This section is always required.
- **Usage section with no commands** — if the reader cannot copy-paste a command and see output, the Usage section has failed.
- **`src/` in Project Layout** — this conflicts with the flat root layout used in all Go and Python projects.
- **Badge soup** — three or four badges maximum. Ten badges on a personal CLI tool is noise.
- **Table of Contents on a short README** — adds scroll overhead with no navigation benefit. Add it only when the file is long enough to need it.

---

## Bilingual Sync (README-PT.md)

`README.md` is the English source of truth. `README-PT.md` is a complete Portuguese translation. Both files must have identical section structure and identical section order at all times.

### Section Heading Translation Reference

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

### What to Translate

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

### Creating README-PT.md from Scratch

1. Copy `README.md` to `README-PT.md`.
2. Translate all section headings using the reference table.
3. Translate all prose.
4. Leave code blocks, commands, paths, and technical terms untouched.
5. Verify structure parity:
   ```bash
   diff <(grep "^[#]" README.md) <(grep "^[#]" README-PT.md)
   ```
   The diff must be empty.

### Syncing After README.md Changes

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

### Translation Quality

- Write clear, professional Brazilian Portuguese
- Match the tone and register of the English version
- Keep sentences direct — Portuguese prose that is shorter than the English is usually better
- Do not add information that is not in the English version
- Do not omit information that is in the English version

### Bilingual Rules

- Never commit one file without the other — always `git add README.md README-PT.md` together
- The English file is always the source of truth — if in doubt, follow the English
- Section order must be identical in both files — the `diff` check is the gate

---

## Related Skills

- **go-skeleton** — Go project layout that the Project Layout section should reflect
- **python-skeleton** — Python project layout that the Project Layout section should reflect
- **makefile-skeleton** — The make targets shown in the Development section
- **github-repo-editor** — Update GitHub repository description and topics after the README is ready
