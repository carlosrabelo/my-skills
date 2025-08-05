---
name: sync-skills
description: Copies all skills from the current project to ~/.claude/skills/, ~/.config/opencode/skills/, ~/.gemini/skills/, ~/.gemini/antigravity/skills/, ~/.qwen/skills/, and ~/.cursor/skills/, excluding itself. Skips destinations that do not exist, except it creates ~/.cursor/skills when ~/.cursor exists. Run this whenever skills are added or updated in the project.
disable-model-invocation: true
mode: manual
category: meta
shared: false
---

# Sync Skills

Execute the bash block in `## Execute` immediately. Do not ask for confirmation, do not summarize the plan — just run it and report the results.

Copy all public skills from this project to the six global destinations. Destinations that do not already exist are skipped, except **`~/.cursor/skills`**: if `~/.cursor` exists and `~/.cursor/skills` does not, the script creates it with `mkdir -p` (Cursor is usually installed but Agent Skills dir may be missing). No other destination directories are created.

## What Gets Synced

Only **public skills** — root-level directories that contain a `SKILL.md`:

```
esp32-skeleton/       git-commit-suggest/   github-repo-editor/
gitignore-skeleton/   go-skeleton/          makefile-skeleton/
monorepo-skeleton/    project-scaffold/     python-skeleton/
readme-skeleton/
```

## What Is NOT Synced

| Path | Reason |
|------|--------|
| `.claude/skills/audit-skills` | Internal — only useful inside this repo |
| `.cursor/skills/audit-skills` | Internal mirror for Cursor — same as `.claude/` counterparts |
| `.claude/skills/sync-skills` | This skill itself |
| `.cursor/skills/sync-skills` | Project-local mirror — not copied by this script |
| `.claude/agents/` | Project-local — system prompts reference this repo specifically |
| `.claude/commands/` | Project-local — commands have hardcoded paths to this repo |

## Destinations

| Destination | Tool |
|-------------|------|
| `~/.claude/skills/` | Claude Code |
| `~/.config/opencode/skills/` | OpenCode |
| `~/.gemini/skills/` | Gemini CLI |
| `~/.gemini/antigravity/skills/` | Antigravity |
| `~/.qwen/skills/` | Qwen |
| `~/.cursor/skills/` | Cursor Agent Skills |

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

1. Determine the project root: `git rev-parse --show-toplevel` from this skill’s directory or the current working directory.
2. Determine the six destinations listed above.
3. Ensure `~/.cursor/skills` exists when `~/.cursor` exists (create if missing). For each destination, check if the directory already exists — skip it if not.
4. Remove all obsolete skill directories from existing destinations.
5. For each root-level directory that contains a `SKILL.md` (excluding `sync-skills`), copy it to all existing destinations.
6. Report which skills were copied and which destinations were skipped.

## Execute

```bash
PROJECT_ROOT="$(git -C "$(dirname "$0")" rev-parse --show-toplevel 2>/dev/null)"
[ -z "$PROJECT_ROOT" ] && PROJECT_ROOT="$(git rev-parse --show-toplevel 2>/dev/null)"
[ -z "$PROJECT_ROOT" ] && PROJECT_ROOT="/home/carlos/Sources/11/my-skills"
DEST_CLAUDE="$HOME/.claude/skills"
DEST_OPENCODE="$HOME/.config/opencode/skills"
DEST_GEMINI="$HOME/.gemini/skills"
DEST_ANTIGRAVITY="$HOME/.gemini/antigravity/skills"
DEST_QWEN="$HOME/.qwen/skills"
DEST_CURSOR="$HOME/.cursor/skills"

if [ -d "$HOME/.cursor" ] && [ ! -d "$DEST_CURSOR" ]; then
  mkdir -p "$DEST_CURSOR"
  echo "+ created: $DEST_CURSOR"
fi

ALL_DESTS=("$DEST_CLAUDE" "$DEST_OPENCODE" "$DEST_GEMINI" "$DEST_ANTIGRAVITY" "$DEST_QWEN" "$DEST_CURSOR")
ACTIVE_DESTS=()
for dest in "${ALL_DESTS[@]}"; do
  if [ -d "$dest" ]; then
    ACTIVE_DESTS+=("$dest")
  else
    echo "- skipped (not found): $dest"
  fi
done

if [ ${#ACTIVE_DESTS[@]} -eq 0 ]; then
  echo "No destinations found. Nothing to sync."
  exit 0
fi

OBSOLETE=(
  go-project go-project-create go-project-migrate
  python-project python-project-create python-project-migrate
  readme-create readme-migrate readme-bilingual
  makefile-create makefile-migrate
  gitignore-create gitignore-migrate
  monorepo-project-create monorepo-project-migrate
)

for name in "${OBSOLETE[@]}"; do
  for dest in "${ACTIVE_DESTS[@]}"; do
    if [ -d "$dest/$name" ]; then
      rm -rf "$dest/$name"
      echo "✗ removed obsolete: $name (from $dest)"
    fi
  done
done

for skill in "$PROJECT_ROOT"/*/; do
  name=$(basename "$skill")
  if [ "$name" != "sync-skills" ] && [ -f "$skill/SKILL.md" ]; then
    for dest in "${ACTIVE_DESTS[@]}"; do
      rm -rf "$dest/$name"
      cp -r "$skill" "$dest/$name"
    done
    echo "✓ $name"
  fi
done
```

## Notes

- Safe to run multiple times — overwrites with the latest version each time.
- Destinations that do not exist are silently skipped — the script does not create them, except `~/.cursor/skills` when `~/.cursor` exists (see `## Execute`).
- Does **not** copy `.claude/agents/` or `.claude/commands/` — they are project-local infrastructure.
- Does **not** delete unrelated skills already present in the destinations.

## Related Skills

- **audit-skills** — Run before syncing to ensure all SKILL.md files are structurally correct.
