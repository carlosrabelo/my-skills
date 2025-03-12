# my-skills

A personal library of reusable Claude Code skills for software development workflows.

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

This repository contains a curated set of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills that encode best practices for software development. Each skill is a detailed guide that Claude invokes during conversations to provide consistent, high-quality assistance across different tasks and projects.

Skills cover Go and Python project structure, Git workflows, code documentation, and GitHub tooling — acting as a shared knowledge base that can be dropped into any project via `sync-skills`.

## Skills

### Git & GitHub

| Skill | Mode | Description |
|-------|------|-------------|
| `git-commit-suggest` | manual | Analyze repository changes and suggest `git add` + `git commit` commands using Conventional Commits formatting |
| `git-workflow-go` | manual | Guide for organizing atomic commits, clean history, and team workflows in Go projects |
| `github-repo-editor` | manual | Generate `gh repo edit` commands with auto-derived description and topics for a repository |

### Go

| Skill | Mode | Description |
|-------|------|-------------|
| `go-project-structure` | agent | Standard Go project layout: `src/` with `go.mod`, dual Makefile hierarchy, `cmd/`, `internal/`, `run/` scripts |
| `go-reorganize-refactor` | agent | Step-by-step patterns for refactoring existing Go code into the standard project structure |
| `go-commenting-en` | agent | Guide for writing clear, idiomatic English comments and godoc documentation in Go code |

### Python

| Skill | Mode | Description |
|-------|------|-------------|
| `python-project-structure` | agent | Standard Python project layout: `src/` layout, `pyproject.toml`, `.venv`, Makefile hierarchy, `run/` scripts |

### Documentation

| Skill | Mode | Description |
|-------|------|-------------|
| `readme-bilingual-sync` | agent | Keep `README.md` (English) and `README-PT.md` (Portuguese) synchronized and up-to-date |

### Meta

| Skill | Mode | Description |
|-------|------|-------------|
| `sync-skills` | manual | Copy all skills from this repository to `~/.claude/skills/` for global use |

**Modes**:
- `manual` — invoked explicitly by the user (e.g. `/git-commit-suggest`)
- `agent` — invoked automatically by Claude when the trigger conditions are met

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- Skills directory at `~/.claude/skills/` (created automatically by Claude Code)

## Installation

Clone the repository and run the sync skill to install all skills globally:

```bash
git clone https://github.com/carlosrabelo/my-skills
cd my-skills
```

Then, inside a Claude Code session in this directory, run:

```
/sync-skills
```

This copies all skills to `~/.claude/skills/`, making them available in every project.

### Manual installation

To install a single skill manually:

```bash
cp -r git-commit-suggest ~/.claude/skills/
```

## Usage

### Invoking a manual skill

In any Claude Code session, type the skill name as a slash command:

```
/git-commit-suggest
/github-repo-editor
/sync-skills
```

### Agent skills

Agent skills are triggered automatically when Claude detects relevant context — for example, `go-project-structure` activates when you're working on a Go project and ask about organization.

### Updating skills

After modifying skills in this repository, re-run `/sync-skills` inside a Claude Code session to push the changes to `~/.claude/skills/`.

## Project Structure

```
my-skills/
├── .claude/
│   └── skills/
│       └── sync-skills/        ← Internal meta-skill (not synced)
│           └── SKILL.md
├── git-commit-suggest/
│   └── SKILL.md
├── git-workflow-go/
│   └── SKILL.md
├── github-repo-editor/
│   └── SKILL.md
├── go-commenting-en/
│   └── SKILL.md
├── go-project-structure/
│   └── SKILL.md
├── go-reorganize-refactor/
│   └── SKILL.md
├── python-project-structure/
│   └── SKILL.md
├── readme-bilingual-sync/
│   └── SKILL.md
├── README.md
└── README-PT.md
```

Each skill lives in its own directory containing a single `SKILL.md` file with the full skill definition, instructions, and examples.

## Adding New Skills

1. Create a new directory with the skill name (use kebab-case):
   ```bash
   mkdir my-new-skill
   ```

2. Create `my-new-skill/SKILL.md` with the skill definition. At minimum include:
   - Frontmatter with `name`, `description`, `category`, `mode`, `shared`
   - An overview section explaining when to use the skill
   - Detailed instructions and examples

3. Run `/sync-skills` in a Claude Code session to install it globally.

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/new-skill-name`
3. Add or update the skill following the structure above
4. Commit using Conventional Commits: `git commit -m "feat: add new-skill-name skill"`
5. Push and open a Pull Request

## License

MIT
