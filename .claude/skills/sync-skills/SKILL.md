---
name: sync-skills
description: Copies all skills from the current project to ~/.claude/skills/ and ~/.config/opencode/skills/, excluding itself. Run this whenever skills are added or updated in the project.
mode: manual
category: meta
shared: false
---

# Sync Skills

Copy all skills from this project to both `~/.claude/skills/` and `~/.config/opencode/skills/`.

## What This Does

The skills in this project live in the **project root** (e.g. `keybindings-help/`, `go-project-structure/`), while `sync-skills` itself lives in `.claude/skills/sync-skills/`. This skill copies every skill directory from the project root into both destinations, skipping `sync-skills`. Any existing symlinks in the opencode destination are removed before copying (to replace the old symlink approach).

## Steps

1. Determine the project root: go up two levels from this SKILL.md file (i.e. `../../` relative to this file, which resolves to the git repo root).
2. Determine the destinations: `~/.claude/skills/` and `~/.config/opencode/skills/`.
3. Create both destination directories if they don't exist.
4. For each subdirectory in the project root that contains a `SKILL.md` (except `sync-skills`), copy it to both destinations.
5. Report which skills were copied and which (if any) were skipped.

## Execute

```bash
PROJECT_ROOT="$(git -C "$(dirname "$0")" rev-parse --show-toplevel 2>/dev/null || echo '/home/carlos/Sources/11/my-skills')"
DEST_CLAUDE="$HOME/.claude/skills"
DEST_OPENCODE="$HOME/.config/opencode/skills"
mkdir -p "$DEST_CLAUDE" "$DEST_OPENCODE"

for skill in "$PROJECT_ROOT"/*/; do
  name=$(basename "$skill")
  if [ "$name" != "sync-skills" ] && [ -f "$skill/SKILL.md" ]; then
    cp -r "$skill" "$DEST_CLAUDE/$name"
    rm -f "$DEST_OPENCODE/$name"
    cp -r "$skill" "$DEST_OPENCODE/$name"
    echo "✓ $name"
  fi
done
```

Or let the agent determine the project root from the working directory and execute the loop above.

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Does **not** delete skills in either destination that no longer exist in the source (non-destructive).
- Does **not** copy `sync-skills` itself — it only makes sense inside a project.
- Removes symlinks in the opencode destination before copying (migrates from the old symlink approach).
