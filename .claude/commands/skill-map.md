---
description: Show current skill inventory with categories, modes and sync status
---
# Skill Map

Show the complete skill inventory for this project.

## Public skills (synced to destinations)

!`for d in /home/carlos/Sources/11/my-skills/*/; do name=$(basename "$d"); if [ -f "$d/SKILL.md" ]; then category=$(grep '^category:' "$d/SKILL.md" 2>/dev/null | head -1 | sed 's/category: //'); mode=$(grep '^mode:' "$d/SKILL.md" 2>/dev/null | head -1 | sed 's/mode: //'); shared=$(grep '^shared:' "$d/SKILL.md" 2>/dev/null | head -1 | sed 's/shared: //'); echo "$name | $category | $mode | shared=$shared"; fi; done | sort`

## Internal skills (not synced)

!`ls /home/carlos/Sources/11/my-skills/.claude/skills/ 2>/dev/null`

## Instructions

Present the data above as a markdown table grouped by category:

```
| Skill | Category | Mode | Shared |
|-------|----------|------|--------|
```

Then list internal skills separately.
Include a count: total public + total internal.
