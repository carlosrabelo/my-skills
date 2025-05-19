---
name: audit-skills
description: Audit all SKILL.md files in this repository for structural deviations and apply auto-fixable corrections.
mode: manual
category: meta
shared: false
---

# Audit Skills

Read every public SKILL.md in this repository, check it against the structural rules below, report all deviations, and apply auto-fixable corrections in place.

## Scope

- Audit target: all `*/SKILL.md` files at the **project root** (one level deep, glob `<root>/*/SKILL.md`)
- Exclude: `.claude/skills/` — internal skills are not in scope
- Determine the project root with `git rev-parse --show-toplevel`

## Checks

Run all six checks on every file. Record findings keyed by the IDs below.

| ID         | Rule |
|------------|------|
| FM-FIELDS  | All five frontmatter fields present: `name`, `description`, `mode`, `category`, `shared` |
| FM-NAME    | `name` value matches the parent directory name exactly |
| FM-VALUES  | `mode` ∈ {agent, manual} · `category` ∈ {git, go, python, tooling, documentation, project, github, meta} · `shared` is a boolean |
| CHANGELOG  | No `## Changelog` section exists anywhere in the file |
| RELATED    | File contains a `## Related Skills` section |
| LANG       | All content (headings and body) is in English — use contextual judgment, not mechanical word-matching |

Additionally, note (informational only, not an error) any `-create` skill missing a matching `-migrate` counterpart, or vice versa.

## Auto-Fix Rules

Apply the following fixes automatically, without asking for confirmation:

- **CHANGELOG** — Remove the `## Changelog` heading and all content under it until the next `##` heading or end of file. Reason: prohibited by project rules.
- **FM-NAME** — Correct the `name` field to match the directory name.
- **RELATED** — If `## Related Skills` is absent, append this stub at the end of the file:

  ```
  ## Related Skills

  - **sync-skills** — Run after changes to propagate corrected skills to all destinations.
  ```

Do **not** auto-fix the following — report only:

- Missing frontmatter fields (cannot infer correct values)
- Invalid `mode` or `category` values (requires human decision)
- Content in Portuguese (requires human translation)

## Steps

1. Determine the project root: `git rev-parse --show-toplevel`.
2. Collect all `*/SKILL.md` at the project root (exclude `.claude/`).
3. For each file, run all six checks and record findings.
4. Print the full audit report (see format below).
5. Apply all auto-fixes in order: CHANGELOG removal → FM-NAME correction → RELATED stub.
6. Print a summary line.
7. Do **not** commit — the user commits manually.

## Report Format

```
AUDIT REPORT
============
git-commit-suggest      OK
go-project-create       OK
go-project-migrate      ISSUES:
  [RELATED] Missing ## Related Skills section → auto-fixed
gitignore-create        ISSUES:
  [CHANGELOG] ## Changelog section found → auto-fixed

NOTE (informational): github-repo-editor has no matching github-repo-migrate.

SUMMARY: 15 audited | 12 clean | 3 issues | 3 auto-fixed | 0 need manual attention
```

- One line per skill: either `OK` or `ISSUES:` followed by indented findings
- Each finding: `[ID] description → auto-fixed` or `[ID] description → manual attention needed`
- Informational pair notes go in a `NOTE` block after the per-skill list
- End with a single `SUMMARY` line

## Notes

- Safe to run multiple times — idempotent.
- Does not commit changes — the user commits manually.
- Skips `.claude/skills/` — internal skills are not audited.

## Related Skills

- **sync-skills** — Run after auditing and fixing to propagate corrected skills to all destinations.
