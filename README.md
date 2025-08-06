# my-skills

A personal library of reusable agent skills for software development workflows (Claude Code, Cursor, OpenCode, Gemini CLI, Antigravity, Qwen).

## Table of Contents

- [Overview](#overview)
- [Skills](#skills)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Adding New Skills](#adding-new-skills)
- [Contributing](#contributing)
- [License](#license)

## Overview

This repository contains a curated set of skills that encode best practices for software development. They are consumed by multiple agent runtimes (Claude Code, Cursor, and other CLIs). Each skill is a detailed guide the agent uses to provide consistent, high-quality assistance across different tasks and projects.

Skills cover Go, C++, and Python project structure, Git workflows, code documentation, and GitHub tooling — acting as a shared knowledge base that can be installed globally via **sync-skills** (six destinations; missing directories are skipped).

## Skills

### Git & GitHub

| Skill | Mode | Description |
|-------|------|-------------|
| `git-commit-suggest` | manual | Analyze repository changes and suggest `git add` + `git commit` commands using Conventional Commits formatting |
| `github-repo-editor` | manual | Generate `gh repo edit` commands with auto-derived description and topics for a repository |
| `gitignore-skeleton` | agent | Standard `.gitignore` structure: secrets, project-type (Go/C++/Python/Node), editors, OS. Creates from scratch or updates existing files. |

### C++

| Skill | Mode | Description |
|-------|------|-------------|
| `cpp-skeleton` | agent | Standard C++ layout: root orchestrator + `projectname/` (`src/`, `tests/`, `lib/`), Makefile + `.make/` scripts, g++, Catch2. Creates from scratch or reorganizes existing projects. |

### Go

| Skill | Mode | Description |
|-------|------|-------------|
| `go-skeleton` | agent | Standard Go project layout: flat root with `go.mod`, single Makefile, `cmd/`, `internal/`, `.make/` scripts. Creates from scratch or reorganizes existing projects. |

### Python

| Skill | Mode | Description |
|-------|------|-------------|
| `python-skeleton` | agent | Standard Python project layout: flat root, `pyproject.toml`, `.venv`, single Makefile, `.make/` scripts. Creates from scratch or reorganizes existing projects. |

### Tooling

| Skill | Mode | Description |
|-------|------|-------------|
| `makefile-skeleton` | agent | Standard Makefile structure: opening lines, self-documenting help, standard targets, `.make/` script delegation. Creates from scratch or updates existing files. |

### Documentation

| Skill | Mode | Description |
|-------|------|-------------|
| `readme-skeleton` | agent | Standard README structure and bilingual sync: section order, Highlights format, Development section, README-PT.md translation. Creates from scratch or updates existing files. |

### Project

| Skill | Mode | Description |
|-------|------|-------------|
| `esp32-skeleton` | agent | Standard ESP32/PlatformIO layout: PlatformIO, `src/`, `.make/` scripts. |
| `monorepo-skeleton` | agent | Standard multi-language monorepo layout: component-level roots, root orchestrator Makefile, consistent naming. |
| `skeleton-scaffold` | agent | Central orchestrator: detects stack, applies one language/platform skeleton, then gitignore-skeleton and readme-skeleton. Detection matrix: [skeleton-scaffold/SKILL.md](skeleton-scaffold/SKILL.md) only (single source of truth). |

### Meta

| Skill | Mode | Description |
|-------|------|-------------|
| `sync-skills` | manual | Copy all public skills from this repository to every configured global skills directory (see [Installation](#installation)) |

**Modes**:
- `manual` — invoked explicitly by the user (e.g. `/git-commit-suggest` in Claude Code)
- `agent` — invoked automatically when the agent detects relevant context

## Prerequisites

- At least one target skills directory (the sync script **does not create** missing destinations):
  - [Claude Code](https://docs.anthropic.com/en/docs/claude-code): `~/.claude/skills/`
  - Cursor: `~/.cursor/skills/` (for global [Agent Skills](https://cursor.com/docs))
  - OpenCode: `~/.config/opencode/skills/`
  - Gemini / Antigravity / Qwen: `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, `~/.qwen/skills/`
- **Note (Cursor):** do not install personal skills under `~/.cursor/skills-cursor/` — that path is reserved for Cursor-built-in skills.

## Installation

Clone the repository and run the sync skill to install all skills globally:

```bash
git clone https://github.com/carlosrabelo/my-skills
cd my-skills
```

Then run the **sync-skills** workflow from this repo (for example `/sync-skills` in Claude Code, or execute the bash block in `.claude/skills/sync-skills/SKILL.md` / `.cursor/skills/sync-skills/SKILL.md`).

That copies every **public** skill (root-level directories with `SKILL.md`) into each **existing** destination: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, `~/.qwen/skills/`, and `~/.cursor/skills/`. If `~/.cursor/` exists but `~/.cursor/skills/` does not, sync-skills creates that folder first; other missing destinations are skipped.

### Manual installation

To install a single skill manually (example paths):

```bash
cp -r git-commit-suggest ~/.claude/skills/
cp -r git-commit-suggest ~/.cursor/skills/
```

## Usage

### Invoking a manual skill

In Claude Code, use slash commands (for example `/git-commit-suggest`). In Cursor, invoke the skill according to your Agent Skills UI or by naming the skill when it applies.

### Agent skills

Agent skills load from context when the agent decides they match — for example, **skeleton-scaffold** when bootstrapping or detecting project type, or a specific **\*-skeleton** (e.g. `go-skeleton`) when the task is already scoped to one stack.

### Updating skills

After modifying skills in this repository, run **sync-skills** again so all existing destinations receive the update.

## Project Structure

```
my-skills/
├── .claude/
│   └── skills/
│       ├── audit-skills/
│       ├── check-sync/
│       └── sync-skills/
│           └── SKILL.md
├── .cursor/
│   └── skills/
│       ├── audit-skills/
│       ├── check-sync/
│       └── sync-skills/
├── cpp-skeleton/
│   └── SKILL.md
├── esp32-skeleton/
│   └── SKILL.md
├── git-commit-suggest/
│   └── SKILL.md
├── github-repo-editor/
│   └── SKILL.md
├── gitignore-skeleton/
│   ├── SKILL.md
│   └── references/
├── go-skeleton/
│   ├── SKILL.md
│   └── references/
├── makefile-skeleton/
│   ├── SKILL.md
│   └── references/
├── monorepo-skeleton/
│   ├── SKILL.md
│   └── references/
├── python-skeleton/
│   ├── SKILL.md
│   └── references/
├── readme-skeleton/
│   ├── SKILL.md
│   └── references/
├── skeleton-scaffold/
│   └── SKILL.md
├── README.md
└── README-PT.md
```

Each skill lives in its own directory. Skills with a `references/` subdirectory split their content across multiple files — `SKILL.md` is the entry point loaded automatically, while files in `references/` are loaded on demand.

## Adding New Skills

1. Create a new directory with the skill name (use kebab-case):
   ```bash
   mkdir my-new-skill
   ```

2. Create `my-new-skill/SKILL.md` with the skill definition. At minimum include:
   - Frontmatter with `name`, `description`, `category`, `mode`, `shared`
   - For Cursor: `description` ≤ 1024 characters; add `disable-model-invocation: true` when `mode` is `manual`
   - An overview section explaining when to use the skill
   - Detailed instructions and examples

3. If the skill is a new **language or platform layout**, add a row to the detection table in **`skeleton-scaffold/SKILL.md`** (single source of truth) and a row in **[AGENTS.md](AGENTS.md)** inventory if it syncs publicly.

4. Run **sync-skills** to install it into every configured global directory.

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/new-skill-name`
3. Add or update the skill following the structure above
4. Commit using Conventional Commits: `git commit -m "feat: add new-skill-name skill"`
5. Push and open a Pull Request

## License

MIT
