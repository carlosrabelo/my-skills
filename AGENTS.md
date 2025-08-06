# my-skills

Central repository for agent skills (Claude Code, Cursor Agent Skills, and other CLIs). All skills are developed and edited here, then synced to six destinations via `/sync-skills`: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, `~/.qwen/skills/`, and `~/.cursor/skills/`. The sync script skips destinations whose directory does not exist, except it creates `~/.cursor/skills/` when `~/.cursor/` exists.

## Rules

- **Edit skills ONLY in this repository.** Never create or edit synced skills directly under global install paths (for example `~/.claude/skills/` or `~/.cursor/skills/`).
- **Do not run `/sync-skills` automatically.** The user runs it manually when they want to sync.
- The `sync-skills` skill (in `.claude/skills/` and mirrored in `.cursor/skills/`) is exclusive to this project and is not copied to global directories as part of public sync.
- **Never add a `## Changelog` section** to skill files — it wastes tokens.
- **All SKILL.md files must be written entirely in English.** These files only become real skills after being synced to the destination directories via `/sync-skills`. Until then, they are source files — write them as if they are already deployed.
- **Conversations are always in Portuguese** — use Portuguese to discuss ideas, plan changes, and explain actions.
- **Cursor:** do not author skills under `~/.cursor/skills-cursor/` — that directory is reserved for Cursor-built-in skills.

## Skill Inventory

**Project detection and which skeleton to run:** defined only in **[skeleton-scaffold/SKILL.md](skeleton-scaffold/SKILL.md)** — update that table when adding stacks; do not duplicate the matrix here.

Public skills (synced to all destinations via `/sync-skills`):

| Skill | Category | Mode | Shared |
|-------|----------|------|--------|
| git-commit-suggest | git | manual | ✓ |
| github-repo-editor | github | manual | ✓ |
| gitignore-skeleton | git | agent | ✓ |
| cpp-skeleton | project | agent | ✓ |
| esp32-skeleton | project | agent | ✓ |
| go-skeleton | go | agent | ✓ |
| makefile-skeleton | tooling | agent | ✓ |
| monorepo-skeleton | project | agent | ✓ |
| skeleton-scaffold | project | agent | ✓ |
| python-skeleton | python | agent | ✓ |
| readme-skeleton | documentation | agent | ✓ |

Internal skills (`.claude/skills/` and `.cursor/skills/` — same content, not synced globally):

| Skill | Purpose |
|-------|---------|
| audit-skills | Audit SKILL.md files in this repo for structural compliance |
| check-sync | Compare source skills against all destinations and report drift |
| sync-skills | Copy public skills to all destination directories |

## Commands

| Command | Purpose |
|---------|---------|
| `/new-skill <name>` | Scaffold a new skill directory with SKILL.md template |
| `/skill-map` | Show current skill inventory with categories and modes |

## Agents

| Agent | Purpose |
|-------|---------|
| `meta-engineer` | Creates and maintains `.claude/` infrastructure and `AGENTS.md` |
| `skill-writer` | Writes and revises SKILL.md files following project conventions |

## Frontmatter Reference

Valid values for SKILL.md frontmatter:

- `mode`: `agent` (LLM runs the skill) or `manual` (explicitly invoked)
- `category`: `git` · `go` · `python` · `tooling` · `documentation` · `project` · `github` · `meta`
- `shared`: `true` (synced to global dirs) or `false` (project-only)
- **`disable-model-invocation`** (Cursor): set `true` when `mode: manual`; **omit** when `mode: agent` so Cursor can auto-load the skill from context
- `description`: must be **≤ 1024 characters** (Cursor limit)
