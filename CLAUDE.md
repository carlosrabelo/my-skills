# my-skills

Central repository for Claude Code skills. All skills are developed and edited here, then synced to `~/.claude/skills/` via `/sync-skills`.

## Rules

- **Edit skills ONLY in this repository.** Never create or edit skills directly in `~/.claude/skills/`.
- **Do not run `/sync-skills` automatically.** The user runs it manually when they want to sync.
- The `sync-skills` skill (in `.claude/skills/`) is exclusive to this project and is not copied to the global directory.
- **Never add a `## Changelog` section** to skill files — it wastes tokens.
- **All SKILL.md files must be written entirely in English.** These files only become real skills after being synced to the destination directories via `/sync-skills`. Until then, they are source files — write them as if they are already deployed.
- **Conversations are always in Portuguese** — use Portuguese to discuss ideas, plan changes, and explain actions.
