---
description: Scaffold a new skill directory with SKILL.md template
---
# New Skill: $ARGUMENTS

Create a new skill named **$ARGUMENTS** in the project root.

## Current skills (to avoid name collisions)

!`ls -d /home/carlos/Sources/11/my-skills/*/ 2>/dev/null | xargs -I{} basename {} | sort`

## Steps

1. Verify `$ARGUMENTS/` does not already exist — abort if it does.
2. Create directory `$ARGUMENTS/` at the project root.
3. Write `$ARGUMENTS/SKILL.md` with the template below, filling `name` with `$ARGUMENTS`.
4. Report the created path.

## Template

```
---
name: $ARGUMENTS
description: TODO — rich description with trigger keywords
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

- Fill `description` with keyword-rich text (this drives automatic trigger)
- Set the correct `category`: `git | go | python | tooling | documentation | project | github | meta`
- Set `mode` to `agent` (LLM runs the skill) or `manual` (human invokes explicitly)
- Add real entries to `## Related Skills`
- Run `/audit` to validate the new file
