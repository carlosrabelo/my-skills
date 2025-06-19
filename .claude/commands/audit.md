---
description: Audit all SKILL.md files in this repository for structural compliance
---
# Audit Skills

Run the **audit-skills** workflow on all public SKILL.md files at the project root.

## Files to audit

!`find /home/carlos/Sources/11/my-skills -maxdepth 2 -name "SKILL.md" | grep -v "\.claude" | sort`

## Instructions

Follow the audit-skills skill exactly:

1. Read each SKILL.md listed above.
2. Run all six checks per file:
   - **FM-FIELDS** — All five frontmatter fields present: `name`, `description`, `mode`, `category`, `shared`
   - **FM-NAME** — `name` value matches the parent directory name exactly
   - **FM-VALUES** — `mode` ∈ {agent, manual} · `category` ∈ {git, go, python, tooling, documentation, project, github, meta} · `shared` is boolean
   - **CHANGELOG** — No `## Changelog` section exists
   - **RELATED** — File contains a `## Related Skills` section
   - **LANG** — All content is in English
3. Apply auto-fixes: remove Changelog, correct FM-NAME, add Related Skills stub.
4. Print the full audit report with per-skill status and a SUMMARY line.
5. Do **not** commit — the user commits manually.
