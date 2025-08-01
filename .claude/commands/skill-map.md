---
description: Show current skill inventory with categories, modes and sync status
---
# Skill Map

Show the complete skill inventory for this project.

## Instructions

1. Use the Glob tool to find all `SKILL.md` files under the project root:
   - Pattern `*/SKILL.md` for public skills (top-level directories)
   - Pattern `.claude/skills/*/SKILL.md` for internal skills (Claude Code)
   - Pattern `.cursor/skills/*/SKILL.md` for internal skills (Cursor — mirror of `.claude/skills/`)

2. For each file found, use Grep to extract the frontmatter fields:
   - `^category:` — the skill category
   - `^mode:` — agent or manual
   - `^shared:` — true or false

3. Present the public skills as a markdown table grouped by category:

```
| Skill | Category | Mode | Shared |
|-------|----------|------|--------|
```

4. List internal skills from `.claude/skills/` and `.cursor/skills/` in separate tables (or one table with a **Location** column: `.claude` vs `.cursor`). Use the `description` frontmatter field as the Purpose. The same three meta-skills should appear under both paths — note if content differs.

5. End with a count line: `Total: X public skills, Y internal skill locations.`
