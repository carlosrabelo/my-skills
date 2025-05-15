---
name: readme-create
description: Reference for creating README.md files from scratch. Covers section order, Highlights format, badge placement, Project Layout block, Development/make-targets section, and what to include or omit per project type. For the Portuguese translation, use readme-bilingual.
mode: agent
category: documentation
shared: true
---

# README Guide

Standard content and structure for project README files.

## Overview

The README is the project's front door. It must work for a first-time visitor who wants to install the tool and for a returning contributor who wants to run the tests. Write for the visitor first — installation and usage before internals.

**Key principle**: Include a section only if it adds value. A shorter README with the right information is better than a padded one with filler sections.

Use this skill when creating README files for the first time. To update existing README files that deviate from the standard, use `readme-migrate`. To create or sync the Portuguese translation, use `readme-bilingual`.

---

## Standard Section Order

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

## Title and Description

```markdown
# Project Name

One sentence describing what the tool does and who it is for.
```

The description line goes immediately below the title, with no heading. Write in plain present tense. Do not start with "A tool that..." — just describe it directly.

✅ `Command-line application that encrypts and decrypts Bitcoin private keys using the BIP38 standard.`

❌ `A tool that can be used to encrypt and decrypt Bitcoin private keys.`

---

## Badges (optional)

Include badges only on public-facing repositories that have CI, a versioned release, or a license worth highlighting. Skip badges on private tools, internal scripts, and monorepo components.

Standard badge trio for Go projects:

```markdown
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/Go-1.25%2B-blue.svg)](https://go.dev/)
[![Release](https://img.shields.io/github/release/username/project-name.svg)](https://github.com/username/project-name/releases)
```

Place badges immediately below the one-line description, before `## Highlights`. Do not add more than three or four badges — badge soup obscures the description.

---

## Highlights

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

## Table of Contents (optional)

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

## Overview (optional)

Include when the one-line description is not enough to orient a new visitor. Keep it to 2–4 sentences covering: what problem does this solve, why does it exist, what makes it different.

Skip it when the project name + one-liner already tells the full story.

---

## Prerequisites

List only genuine prerequisites — things a user might not have and must install before the tool works. Do not list git, make, or other universal tools.

```markdown
## Prerequisites

- **Go 1.25+** — required to build from source; [download](https://go.dev/dl/)
- **golangci-lint** — required for `make lint`; install with `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
```

Bold the tool name, follow with the minimum version and a one-line install note when it is not obvious.

---

## Installation

Always include the "Build from Source" path — it works for every environment. Optional additional paths: `go install`, pre-built binary download.

```markdown
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
```

Install path convention: `~/.local/bin` for user installs, `/usr/local/bin` for system installs. For Python projects, `make setup` goes before `make build`.

---

## Quick Start (optional)

The minimum path from nothing to first successful output. Under 10 lines including bash blocks. Merge into Installation when the project is very simple.

```markdown
## Quick Start

```bash
make build
./bin/project-name --help
```
```

---

## Usage

Show real command examples with real flags. Each example should be self-contained — do not require readers to have run a previous example.

```markdown
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
```

Show actual output when it clarifies what the command does. Use subheadings for distinct modes or command groups.

❌ Do not write a Usage section that is only prose with no commands.

---

## Configuration (optional)

Include only when the project has a config file or significant environment variables. Skip for simple CLI tools that only have flags.

Show the actual config file format with comments:

```markdown
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
```

---

## Project Layout (optional)

Include when the directory structure is non-obvious or when contributors need orientation. Show only meaningful top-level entries with a one-line comment per entry — do not list every file.

Use the section name `## Project Layout` (not `## Project Structure`):

```markdown
## Project Layout

```
cmd/project-name/    # Go entry point
internal/domain/     # Core business logic
internal/cli/        # Command definitions and user interaction
bin/                 # Compiled binaries (git-ignored)
make/                 # Build and install scripts
cfg/                 # Default configuration files
```
```

❌ Do not show `src/` — it conflicts with the flat root layout used in all Go and Python projects.

---

## Development

Always include this section. It shows the make targets a contributor needs to know:

```markdown
## Development

```bash
make build      # Compile binary to bin/project-name
make test       # Run all tests
make quality    # Format, vet, and lint
make install    # Install to ~/.local/bin
```
```

For Python projects, add `make setup` as the first step:

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

## Contributing (optional)

Include for public open-source projects. Skip for personal tools and internal scripts. Keep it brief:

```markdown
## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/description`
3. Commit with Conventional Commits: `git commit -m "feat: add X"`
4. Push and open a pull request
```

---

## License

Always the last section. One line referencing the LICENSE file:

```markdown
## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
```

---

## Minimal Complete Example

Full README.md for a simple Go CLI tool — start here and add sections as needed:

```markdown
# project-name

One sentence describing what the tool does.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Highlights

- Does X in a Y way
- Supports Z
- Works on Linux, macOS, and Windows

## Prerequisites

- **Go 1.25+** — [download](https://go.dev/dl/)

## Installation

```bash
git clone https://github.com/username/project-name.git
cd project-name
make build
make install
```

## Usage

```bash
project-name --flag value
```

## Development

```bash
make build      # Compile binary
make test       # Run all tests
make quality    # Format, vet, and lint
```

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
```

---

## Anti-Patterns

❌ **"A tool that..." in the description** — describes the category, not the tool. Write what it does.

❌ **Mismatched section structure between README.md and README-PT.md** — the two files must have identical section order. If one has `## Configuration`, the other must too.

❌ **Omitting `## Development`** — every contributor needs to know how to build and test. This section is always required.

❌ **Usage section with no commands** — if the reader cannot copy-paste a command and see output, the Usage section has failed.

❌ **`src/` in Project Layout** — this conflicts with the flat root layout used in all Go and Python projects.

❌ **Badge soup** — three or four badges maximum. Ten badges on a personal CLI tool is noise.

❌ **Table of Contents on a short README** — adds scroll overhead with no navigation benefit. Add it only when the file is long enough to need it.

---

## Key Principles

- Bilingual always: `README.md` (English) and `README-PT.md` (Portuguese), identical structure
- Section order: Highlights → Prerequisites → Installation → Usage → Development → License
- Plain bullets in Highlights — no trailing periods, no bold labels
- `## Development` always shows the make targets; never omit it
- Include a section only when it adds real value — prefer a shorter README

---

## Related Skills

- `readme-bilingual` — create or update the Portuguese translation after writing README.md
- `readme-migrate` — bring an existing README.md up to this standard
- `makefile-create` — the make targets shown in the Development section
- `go-project-create` — the directory layout that Project Layout should reflect for Go projects
- `github-repo-editor` — update GitHub repository description and topics after the README is ready
- `python-project-create` — same, for Python projects
