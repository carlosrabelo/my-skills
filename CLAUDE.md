# my-skills

Central repository for Claude Code skills. All skills are developed and edited here, then synced to five destinations via `/sync-skills`: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, `~/.gemini/antigravity/skills/`, and `~/.qwen/scripts/`.

## Rules

- **Edit skills ONLY in this repository.** Never create or edit skills directly in `~/.claude/skills/`.
- **Do not run `/sync-skills` automatically.** The user runs it manually when they want to sync.
- The `sync-skills` skill (in `.claude/skills/`) is exclusive to this project and is not copied to the global directory.
- **Never add a `## Changelog` section** to skill files — it wastes tokens.
- **All SKILL.md files must be written entirely in English.** These files only become real skills after being synced to the destination directories via `/sync-skills`. Until then, they are source files — write them as if they are already deployed.
- **Conversations are always in Portuguese** — use Portuguese to discuss ideas, plan changes, and explain actions.

## Skill Inventory

Public skills (synced to all destinations via `/sync-skills`):

| Skill | Category | Mode | Shared |
|-------|----------|------|--------|
| git-commit-suggest | git | manual | ✓ |
| github-repo-editor | github | manual | ✓ |
| gitignore-skeleton | git | agent | ✓ |
| esp32-skeleton | project | agent | ✓ |
| go-skeleton | go | agent | ✓ |
| makefile-skeleton | tooling | agent | ✓ |
| monorepo-skeleton | project | agent | ✓ |
| project-scaffold | project | agent | ✓ |
| python-skeleton | python | agent | ✓ |
| readme-skeleton | documentation | agent | ✓ |

Internal skills (`.claude/skills/` — not synced):

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
| `meta-engineer` | Creates and maintains `.claude/` infrastructure and `CLAUDE.md` |
| `skill-writer` | Writes and revises SKILL.md files following project conventions |

## Frontmatter Reference

Valid values for SKILL.md frontmatter:

- `mode`: `agent` (LLM runs the skill) or `manual` (explicitly invoked)
- `category`: `git` · `go` · `python` · `tooling` · `documentation` · `project` · `github` · `meta`
- `shared`: `true` (synced to global dirs) or `false` (project-only)
