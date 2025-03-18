---
name: sync-skills
description: Copies all skills from the current project's skills directory to ~/.claude/skills/, excluding itself. Run this whenever skills are added or updated in the project.
mode: manual
category: meta
shared: false
---

# Sync Skills

Copy all skills from this project to the global Claude skills directory.

## What This Does

The skills in this project live in the **project root** (e.g. `keybindings-help/`, `go-project-structure/`), while `sync-skills` itself lives in `.claude/skills/sync-skills/`. This skill copies every skill directory from the project root into `~/.claude/skills/`, skipping `sync-skills`.

## Steps

1. Determine the project root: go up two levels from this SKILL.md file (i.e. `../../` relative to this file, which resolves to the git repo root).
2. Determine the destination: `~/.claude/skills/`.
3. For each subdirectory in the project root that contains a `SKILL.md` (except `sync-skills`), run:
   ```bash
   cp -r <project-root>/<skill-name> ~/.claude/skills/
   ```
4. Report which skills were copied and which (if any) were skipped.

## Execute

```bash
PROJECT_ROOT="$(git -C "$(dirname "$0")" rev-parse --show-toplevel 2>/dev/null || echo '/home/carlos/Sources/11/my-skills')"
for skill in "$PROJECT_ROOT"/*/; do
  name=$(basename "$skill")
  if [ "$name" != "sync-skills" ] && [ -f "$skill/SKILL.md" ]; then
    cp -r "$skill" ~/.claude/skills/"$name"
    echo "✓ $name"
  fi
done
```

Or let the agent determine the project root from the working directory and execute the loop above.

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Does **not** delete skills in `~/.claude/skills/` that no longer exist in the source (non-destructive).
- Does **not** copy `sync-skills` itself to the global directory — it only makes sense inside a project.
