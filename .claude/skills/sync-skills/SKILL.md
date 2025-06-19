---
name: sync-skills
description: Copies all skills from the current project to ~/.claude/skills/, ~/.config/opencode/skills/, ~/.gemini/skills/, and ~/.gemini/antigravity/skills/, excluding itself. Run this whenever skills are added or updated in the project.
mode: manual
category: meta
shared: false
---

# Sync Skills

Copy all public skills from this project to the four global destinations.

## What Gets Synced

Only **public skills** — root-level directories that contain a `SKILL.md`:

```
git-commit-suggest/   github-repo-editor/   gitignore-skeleton/
go-skeleton/          makefile-skeleton/    monorepo-skeleton/
python-skeleton/      readme-skeleton/
```

## What Is NOT Synced

| Path | Reason |
|------|--------|
| `.claude/skills/audit-skills` | Internal — only useful inside this repo |
| `.claude/skills/sync-skills` | This skill itself |
| `.claude/agents/` | Project-local — system prompts reference this repo specifically |
| `.claude/commands/` | Project-local — commands have hardcoded paths to this repo |

## Destinations

| Destination | Tool |
|-------------|------|
| `~/.claude/skills/` | Claude Code |
| `~/.config/opencode/skills/` | OpenCode |
| `~/.gemini/skills/` | Gemini CLI |
| `~/.gemini/antigravity/skills/` | Antigravity |

## Obsolete Skills

These skill names have been renamed or merged. Remove them from all destinations before syncing:

```
go-project
go-project-create
go-project-migrate
python-project
python-project-create
python-project-migrate
readme-create
readme-migrate
readme-bilingual
makefile-create
makefile-migrate
gitignore-create
gitignore-migrate
monorepo-project-create
monorepo-project-migrate
```

## Steps

1. Determine the project root: go up two levels from this file (`../../` → git repo root).
2. Determine the four destinations listed above.
3. Create all destination directories if they don't exist.
4. Remove all obsolete skill directories from every destination.
5. For each root-level directory that contains a `SKILL.md` (excluding `sync-skills`), copy it to all four destinations.
6. Report which skills were copied and which (if any) were skipped.

## Execute

```bash
PROJECT_ROOT="$(git -C "$(dirname "$0")" rev-parse --show-toplevel 2>/dev/null || echo '/home/carlos/Sources/11/my-skills')"
DEST_CLAUDE="$HOME/.claude/skills"
DEST_OPENCODE="$HOME/.config/opencode/skills"
DEST_GEMINI="$HOME/.gemini/skills"
DEST_ANTIGRAVITY="$HOME/.gemini/antigravity/skills"
mkdir -p "$DEST_CLAUDE" "$DEST_OPENCODE" "$DEST_GEMINI" "$DEST_ANTIGRAVITY"

OBSOLETE=(
  go-project go-project-create go-project-migrate
  python-project python-project-create python-project-migrate
  readme-create readme-migrate readme-bilingual
  makefile-create makefile-migrate
  gitignore-create gitignore-migrate
  monorepo-project-create monorepo-project-migrate
)

for name in "${OBSOLETE[@]}"; do
  for dest in "$DEST_CLAUDE" "$DEST_OPENCODE" "$DEST_GEMINI" "$DEST_ANTIGRAVITY"; do
    if [ -d "$dest/$name" ]; then
      rm -rf "$dest/$name"
      echo "✗ removed obsolete: $name (from $dest)"
    fi
  done
done

for skill in "$PROJECT_ROOT"/*/; do
  name=$(basename "$skill")
  if [ "$name" != "sync-skills" ] && [ -f "$skill/SKILL.md" ]; then
    for dest in "$DEST_CLAUDE" "$DEST_OPENCODE" "$DEST_GEMINI" "$DEST_ANTIGRAVITY"; do
      rm -rf "$dest/$name"
      cp -r "$skill" "$dest/$name"
    done
    echo "✓ $name"
  fi
done
```

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Does **not** copy `.claude/agents/` or `.claude/commands/` — they are project-local infrastructure.
- Does **not** delete unrelated skills already present in the destinations.

## Related Skills

- **audit-skills** — Run before syncing to ensure all SKILL.md files are structurally correct.
