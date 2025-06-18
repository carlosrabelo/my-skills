---
name: sync-skills
description: Copies all skills from the current project to ~/.claude/skills/, ~/.config/opencode/skills/, ~/.gemini/skills/, and ~/.gemini/antigravity/skills/, excluding itself. Run this whenever skills are added or updated in the project.
mode: manual
category: meta
shared: false
---

# Sync Skills

Copy all skills from this project to `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, and `~/.gemini/antigravity/skills/`.

## What This Does

The skills in this project live in the **project root** (e.g. `keybindings-help/`, `go-skeleton/`), while `sync-skills` itself lives in `.claude/skills/sync-skills/`. This skill:

1. **Removes obsolete skill directories** from all destinations (skills that have been renamed or deleted)
2. **Copies every current skill** from the project root into all four destinations

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

1. Determine the project root: go up two levels from this SKILL.md file (i.e. `../../` relative to this file, which resolves to the git repo root).
2. Determine the destinations: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, and `~/.gemini/antigravity/skills/`.
3. Create all destination directories if they don't exist.
4. Remove all obsolete skill directories (listed above) from every destination.
5. For each subdirectory in the project root that contains a `SKILL.md` (except `sync-skills`), copy it to all destinations.
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
    rm -rf "$DEST_CLAUDE/$name"
    cp -r "$skill" "$DEST_CLAUDE/$name"
    rm -rf "$DEST_OPENCODE/$name"
    cp -r "$skill" "$DEST_OPENCODE/$name"
    rm -rf "$DEST_GEMINI/$name"
    cp -r "$skill" "$DEST_GEMINI/$name"
    rm -rf "$DEST_ANTIGRAVITY/$name"
    cp -r "$skill" "$DEST_ANTIGRAVITY/$name"
    echo "✓ $name"
  fi
done
```

Or let the agent determine the project root from the working directory and execute the steps above.

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Removes obsolete skill names from all destinations before copying (cleans up renamed/merged skills).
- Does **not** delete skills in any destination that are not in the obsolete list and do not exist in this project (non-destructive for unrelated skills).
- Does **not** copy `sync-skills` itself — it only makes sense inside a project.
