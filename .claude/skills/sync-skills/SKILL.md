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

The skills in this project live in the **project root** (e.g. `keybindings-help/`, `go-project-structure/`), while `sync-skills` itself lives in `.claude/skills/sync-skills/`. This skill copies every skill directory from the project root into all four destinations, skipping `sync-skills`. Any existing skill directories in the destinations are removed before copying.

## Steps

1. Determine the project root: go up two levels from this SKILL.md file (i.e. `../../` relative to this file, which resolves to the git repo root).
2. Determine the destinations: `~/.claude/skills/`, `~/.config/opencode/skills/`, `~/.gemini/skills/`, and `~/.gemini/antigravity/skills/`.
3. Create all destination directories if they don't exist.
4. For each subdirectory in the project root that contains a `SKILL.md` (except `sync-skills`), copy it to all destinations.
5. Report which skills were copied and which (if any) were skipped.

## Execute

```bash
PROJECT_ROOT="$(git -C "$(dirname "$0")" rev-parse --show-toplevel 2>/dev/null || echo '/home/carlos/Sources/11/my-skills')"
DEST_CLAUDE="$HOME/.claude/skills"
DEST_OPENCODE="$HOME/.config/opencode/skills"
DEST_GEMINI="$HOME/.gemini/skills"
DEST_ANTIGRAVITY="$HOME/.gemini/antigravity/skills"
mkdir -p "$DEST_CLAUDE" "$DEST_OPENCODE" "$DEST_GEMINI" "$DEST_ANTIGRAVITY"

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

Or let the agent determine the project root from the working directory and execute the loop above.

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Does **not** delete skills in any destination that no longer exist in the source (non-destructive).
- Does **not** copy `sync-skills` itself — it only makes sense inside a project.
- Removes existing skill directories in all destinations before copying (handles both symlinks and directories).
