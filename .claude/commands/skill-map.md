---
description: Show current skill inventory with categories, modes and sync status
---
# Skill Map

Show the complete skill inventory for this project.

## Instructions

1. Use the Glob tool to find all `SKILL.md` files under the project root:
   - Pattern `*/SKILL.md` for public skills (top-level directories)
   - Pattern `.claude/skills/*/SKILL.md` for internal skills

2. For each file found, use Grep to extract the frontmatter fields:
   - `^category:` — the skill category
   - `^mode:` — agent or manual
   - `^shared:` — true or false

3. Present the public skills as a markdown table grouped by category:

```
| Skill | Category | Mode | Shared |
|-------|----------|------|--------|
```

4. List internal skills (`.claude/skills/`) in a separate table with Skill and Purpose columns. Use the `description` frontmatter field as the Purpose.

5. End with a count line: `Total: X public skills, Y internal skills.`
