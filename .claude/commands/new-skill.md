---
description: Scaffold a new skill directory with SKILL.md template
---
# New Skill: $ARGUMENTS

Create a new skill named **$ARGUMENTS** in the project root.

## Current skills (to avoid name collisions)

!`cd "$(git rev-parse --show-toplevel 2>/dev/null || pwd)" && ls -d */ 2>/dev/null | xargs -n1 basename | sort`

## Steps

1. Verify `$ARGUMENTS/` does not already exist — abort if it does.
2. Create directory `$ARGUMENTS/` at the project root.
3. Write `$ARGUMENTS/SKILL.md` with the template below, filling `name` with `$ARGUMENTS`.
4. Report the created path.

## Template

```
---
name: $ARGUMENTS
description: TODO — rich description with trigger keywords (max 1024 chars for Cursor)
mode: agent
category: tooling
shared: true
---

# TODO: Skill Title

TODO: One-sentence purpose statement.

## Context Detection

TODO: Describe how to detect create vs migrate flow (if applicable).

## Steps

1. TODO

## Related Skills

- **sync-skills** — Run after changes to propagate to all destinations.
```

## After creating, remind the user to

- Fill `description` with keyword-rich text (this drives automatic trigger); keep it **≤ 1024 characters** (Cursor Agent Skills limit)
- Set the correct `category`: `git | go | python | tooling | documentation | project | github | meta`
- Set `mode` to `agent` (LLM runs the skill) or `manual` (human invokes explicitly)
- **Cursor alignment:** for `mode: manual`, add `disable-model-invocation: true` to the frontmatter; for `mode: agent`, omit `disable-model-invocation` so the agent can auto-invoke from context
- Add real entries to `## Related Skills`
- Run `/audit` to validate the new file
