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

Copies every skill directory found alongside this one (i.e. siblings of `sync-skills/`) into `~/.claude/skills/`, overwriting any existing version. Skips `sync-skills` itself.

## Steps

1. Determine the source directory: the parent of this SKILL.md file.
2. Determine the destination: `~/.claude/skills/`.
3. For each subdirectory in source (except `sync-skills`), run:
   ```bash
   cp -r <source>/<skill-name> ~/.claude/skills/
   ```
4. Report which skills were copied and which (if any) were skipped.

## Execute

Run the following, replacing `<SKILLS_DIR>` with the absolute path to this project's skills directory:

```bash
for skill in <SKILLS_DIR>/*/; do
  name=$(basename "$skill")
  if [ "$name" != "sync-skills" ]; then
    cp -r "$skill" ~/.claude/skills/"$name"
    echo "✓ $name"
  fi
done
```

Or let the agent determine `<SKILLS_DIR>` automatically from the location of this file and execute the loop above.

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Does **not** delete skills in `~/.claude/skills/` that no longer exist in the source (non-destructive).
- Does **not** copy `sync-skills` itself to the global directory — it only makes sense inside a project.
