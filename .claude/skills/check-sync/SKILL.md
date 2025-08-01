---
name: check-sync
description: Compare public skills in the source repo against all six sync destinations and report drift. Shows OK, DRIFT, or MISSING per skill per destination.
disable-model-invocation: true
mode: manual
category: meta
shared: false
---

# Check Sync

Execute the bash block in `## Execute` immediately. Do not ask for confirmation, do not summarize the plan — just run it and report the results.

Compare each public skill in the source repo against all six sync destinations and report their sync status.

## Status Codes

| Code | Meaning |
|------|---------|
| `OK` | Destination copy is identical to source |
| `DRIFT` | Destination copy exists but content differs |
| `MISSING` | Skill does not exist at the destination |

## Scope

- **Source**: root-level directories containing a `SKILL.md` (excludes `.claude/`)
- **Destinations**: same six targets as `sync-skills`

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

issues=()
total=0

echo "SYNC STATUS REPORT"
echo "=================="

for skill in "$PROJECT_ROOT"/*/; do
  name=$(basename "$skill")
  [[ "$name" == .* ]] && continue
  [ ! -f "$skill/SKILL.md" ] && continue

  total=$((total + 1))
  statuses=()
  skill_issues=()

  for dest_pair in "claude:$DEST_CLAUDE" "opencode:$DEST_OPENCODE" "gemini:$DEST_GEMINI" "antigravity:$DEST_ANTIGRAVITY" "qwen:$DEST_QWEN" "cursor:$DEST_CURSOR"; do
    dest_name="${dest_pair%%:*}"
    dest_path="${dest_pair#*:}"

    if [ ! -d "$dest_path/$name" ]; then
      statuses+=("${dest_name}=MISSING")
      skill_issues+=("MISSING: ${dest_name}/${name}")
    elif ! diff -rq "$skill" "$dest_path/$name" > /dev/null 2>&1; then
      statuses+=("${dest_name}=DRIFT")
      skill_issues+=("DRIFT:   ${dest_name}/${name}")
    else
      statuses+=("${dest_name}=OK")
    fi
  done

  issues+=("${skill_issues[@]}")
  printf "  %-26s  %s\n" "$name" "${statuses[*]}"
done

echo ""
if [ ${#issues[@]} -eq 0 ]; then
  echo "SUMMARY: ${total} skills | all in sync"
else
  sync_count=$((total - $(printf '%s\n' "${issues[@]}" | grep -c .) ))
  echo "SUMMARY: ${total} skills | ${#issues[@]} issue(s) found"
  echo ""
  for issue in "${issues[@]}"; do
    echo "  $issue"
  done
  echo ""
  echo "Run /sync-skills to fix."
fi
```

## Notes

- Safe to run multiple times — read-only, makes no changes.
- Does not check skills inside `.claude/skills/` or `.cursor/skills/` (internal skills are not synced).
- DRIFT is detected via `diff -rq` (recursive content comparison).

## Related Skills

- **sync-skills** — Run to push source skills to all destinations and resolve MISSING or DRIFT.
- **audit-skills** — Run before syncing to catch structural issues in the source first.
