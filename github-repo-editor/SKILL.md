---
name: github-repo-editor
description: Analyze project code and generate a gh repo edit command with proper description and topics. Use this whenever you want to update your GitHub repository metadata (description and topics). Analyzes project functionality to create accurate, concise descriptions (max 350 chars) and selects 3-5 relevant topics. Returns copy-paste ready gh commands.
category: github
mode: manual
shared: true
---

# GitHub Repository Editor Skill

Generate gh repo edit commands to update repository description and topics.

## Overview

This skill analyzes your project code and generates ready-to-use `gh repo edit` commands that update your GitHub repository metadata with accurate descriptions and relevant topics.

## When to Use

Use this skill when:
- Creating a new GitHub repository for a project
- Updating repository metadata and visibility
- Adding or improving repository description
- Setting repository topics for discoverability
- Preparing a project for public release

## Analysis Process

### 1. Code Analysis
- Examine project source code
- Understand primary functionality
- Identify key features and purpose
- Review README and documentation
- Determine project type and scope

### 2. Description Generation
- Create concise, clear project description
- Highlight main functionality
- Include key features if space allows
- Keep under 350 characters (GitHub limit)
- Write in English only

### 3. Topic Selection
- Identify 3-5 most relevant topics
- Consider primary language/technology
- Include problem domain (e.g., cli, game, api)
- Add relevant technologies used
- Choose from GitHub standard topics when possible

### 4. Command Generation
- Format `gh repo edit` command
- Include description flag with proper quoting
- Add topics in comma-separated format
- Format for copy-paste execution

## Description Guidelines

### Length
- **Maximum**: 350 characters (GitHub limit)
- **Recommended**: 150-250 characters for clarity
- **Minimum**: 50 characters (descriptive enough)

### Content
✅ **Good**:
- Clear, direct description of what the project does
- Primary use case or problem solved
- Key technology or approach if relevant
- Professional, concise language

❌ **Avoid**:
- Vague or generic descriptions
- Excessive superlatives ("amazing", "best")
- URLs or links (use topics instead)
- Abbreviations without context
- Marketing-speak

### Examples

**Short Project**:
> CLI tool for detecting package inconsistencies across Linux systems and suggesting installation commands

**With Technologies**:
> Multiplayer space trading game with turn-based combat, player-driven economy, and cosmetics monetization. Built with Go, Svelte, PostgreSQL.

**Problem-Focused**:
> Identifies package version mismatches across development machines and provides installation commands for APT, SNAP, and Flatpak.

## Topics Guidelines

### Topic Selection
- Choose 3-5 topics maximum
- Use lowercase with hyphens
- Prefer GitHub standard topics
- Relevance over quantity

### Common Topics

**By Type**:
- `cli` — Command-line tool
- `api` — API service
- `library` — Reusable library
- `framework` — Framework or toolkit
- `game` — Game project
- `web-app` — Web application
- `desktop-app` — Desktop application

**By Language**:
- `go` — Go/Golang
- `python` — Python
- `javascript` — JavaScript
- `typescript` — TypeScript
- `shell` — Shell scripting

**By Domain**:
- `package-management` — Package manager related
- `devops` — DevOps tools
- `monitoring` — Monitoring/observability
- `testing` — Testing tools
- `documentation` — Documentation tools

**By Purpose**:
- `automation` — Task automation
- `productivity` — Productivity tools
- `utilities` — General utilities
- `developer-tools` — Developer-focused

### Examples

**Makalu CLI Tool**:
- Topics: `cli`, `go`, `package-management`, `linux`, `devops`

**Void Traders Game**:
- Topics: `game`, `go`, `multiplayer`, `turn-based`, `svelte`

**API Backend Service**:
- Topics: `api`, `golang`, `web`, `backend`, `postgresql`

**Python Testing Framework**:
- Topics: `python`, `testing`, `framework`, `pytest`, `utilities`

## Command Format

### Basic Format
```bash
gh repo edit --description "Your description here" --add-topic "topic1,topic2,topic3"
```

### With Full Details
```bash
gh repo edit \
  --description "Clear, concise description under 350 chars" \
  --add-topic "topic1,topic2,topic3,topic4,topic5"
```

### Important Notes
- All text **must be in English**
- Description must be wrapped in quotes
- Topics are comma-separated (no spaces after commas)
- Maximum 5 topics (GitHub recommendation)
- Topics should be lowercase and use hyphens for spaces

## Execution Requirements

⚠️ **Important**:
- ❌ Never execute the gh command automatically
- ✅ Return command as plain text for copy-paste
- ✅ Ensure command is complete and syntactically correct
- ✅ Include proper quoting and escaping
- ✅ Format for easy terminal use

## Output Format

The skill returns:
1. **Project Analysis**: Brief summary of project
2. **Suggested Description**: The description text (will fit in 350 chars)
3. **Character Count**: Display count to show it fits
4. **Selected Topics**: List of 3-5 topics with explanation
5. **Command**: Complete `gh repo edit` command ready to copy

### Example Output

```
## Project Analysis
Makalu is a CLI tool for detecting package inconsistencies across Ubuntu machines.

## Suggested Description
CLI tool for detecting package inconsistencies across Linux systems. Suggests installation commands for APT, SNAP, and Flatpak package managers.

## Character Count
98 / 350 characters ✓

## Selected Topics
- cli (command-line tool)
- go (primary language)
- package-management (domain)
- linux (platform)
- devops (use case)

## GitHub Command
gh repo edit --description "CLI tool for detecting package inconsistencies across Linux systems. Suggests installation commands for APT, SNAP, and Flatpak." --add-topic "cli,go,package-management,linux,devops"
```

## Usage Steps

1. **Provide project context**: Share code files or project overview
2. **Skill analyzes**: Examines functionality and structure
3. **Returns command**: Shows complete gh command
4. **You copy**: Copy the command from the output
5. **You paste**: Run in your terminal (requires `gh` CLI installed)

## Prerequisites

- GitHub CLI installed: `brew install gh` (macOS) or equivalent
- Authenticated with GitHub: `gh auth login`
- Write permissions on the repository
- Repository must be cloned or have `gh` access

## Testing the Command

```bash
# Test what would change without applying
gh repo view

# Apply the changes
gh repo edit --description "..." --add-topic "..."

# Verify changes
gh repo view --web  # Opens on GitHub
```

---

## Related Skills

- **readme-bilingual-sync** — For creating comprehensive project documentation
- **git-commit-suggest** — For committing changes after updating repository metadata
- **go-project-structure** — For documenting project structure

## Changelog

- **v1.0** (2025-03-12) — Initial shared version
  - Generates gh repo edit commands with description and topics
  - Description guidelines (max 350 chars)
  - Topic selection guidelines (3-5 topics)
  - Output format with project analysis summary
