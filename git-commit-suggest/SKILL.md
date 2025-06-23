---
name: git-commit-suggest
description: Manually invoked to suggest git add and git commit commands with Conventional Commits formatting.
mode: manual
category: git
shared: true
---

# Git Commit Suggest

Analyze current repository changes and suggest a single `git add` + `git commit` command pair, ready to copy-paste.

## Behavior

- **Only suggest commands — never execute them.**
- Always produce exactly **one** `git add` command and **one** `git commit` command.
- Output only the two commands in a code block, nothing else after them.

## Process

1. Run `git status` to see what changed.
2. Run `git diff` and `git diff --staged` to understand the changes.
3. Identify files that should be committed (skip secrets, build artifacts, lock files unless explicitly relevant).
4. Choose the right Conventional Commits type based on the changes.
5. Output the two commands.

## Conventional Commits Format

```
type(scope): short description in English
```

Types: `feat` · `fix` · `refactor` · `docs` · `test` · `chore` · `style` · `perf`

Use `-m "body"` only when extra context is genuinely needed.

## Output Format

Show a brief summary of what changed (2–4 bullet points), then the two commands:

```bash
git add <file1> <file2> ...
git commit -m "type(scope): description"
```

If a body is needed:

```bash
git add <file1> <file2> ...
git commit -m "type(scope): description" -m "body explaining why"
```

No other text after the code block.

## Related Skills

- **sync-skills** — Run after changes to propagate corrected skills to all destinations.
