---
name: git-commit-suggest
description: Manually invoked to suggest git add and git commit commands with Conventional Commits formatting.
mode: manual
category: git
shared: true
---

# Git Commit Suggest

Analyze current repository changes and suggest properly formatted git commands ready to copy-paste or execute.

## Overview

This agent/skill provides complete analysis of your staged and unstaged changes, validates them against project conventions, and suggests git commands that follow Conventional Commits standards.

## When to Use

### 🔵 Interactive Mode (Claude, etc.)
Use this skill whenever you:
- Have changes ready to commit and want suggested commands
- Need to validate that changes align with your commit rules
- Want to identify files that shouldn't be committed
- Need proper Conventional Commits formatting

### 🟠 Agent/Automation Mode (OpenCode, etc.)
Invoke with:
```
@git-commit-suggest
@git-commit-suggest analyze
@git-commit-suggest validate
```

## Analysis Process

### 1. Repository Analysis
- Check current status (`git status`)
- Analyze staged changes (`git diff --staged`)
- Analyze unstaged changes (`git diff`)
- Identify modified/created/deleted files

### 2. Validation Rules
- Conform to Conventional Commits standard
- Validate commit type (feat, fix, refactor, docs, test, chore, style, perf)
- Check for blocked files that shouldn't be committed
- Analyze if dependencies or tests need updates

### 3. Preparation Suggestions
- Suggest which files to add (`git add`)
- Identify potential commit issues
- Recommend additional checks if needed

### 4. Command Generation
- Generate complete, validated git commit command
- Include commit body if appropriate
- Format for interactive use or automated agent pipelines

## Conventional Commits Format
```bash
git commit -m "type(scope): short description in English" -m "detailed body if needed"
```

### Commit Types
- `feat` — New feature
- `fix` — Bug fix
- `refactor` — Code refactoring
- `docs` — Documentation
- `test` — Tests
- `chore` — Maintenance tasks
- `style` — Formatting changes
- `perf` — Performance improvements

### Example
```bash
git commit -m "feat(makalu): add package diff detection" -m "Implements intelligent package comparison across systems using APT, SNAP, and Flatpak managers"
```

## Output Format

### For Claude
**Summary of Changes Found**:
```
- Modified: cmd/makalu/main.go
- Modified: pkg/discovery/discovery.go
- Created: tests/discovery_test.go
```

**Validation Results**:
```
✓ All files allowed for commit
✓ Changes align with project structure
⚠️ Consider adding unit test execution before committing
```

**Suggested Commands** (ready to copy-paste):
```bash
git add cmd/makalu/main.go pkg/discovery/discovery.go tests/discovery_test.go
git commit -m "feat(makalu): implement package discovery module" -m "Adds intelligent detection across APT, SNAP, Flatpak with comprehensive error handling and test coverage"
```

### For Agent/Automation Mode
Can return as:
- JSON for pipeline processing
- Shell commands for direct execution
- Structured report for logging

## Security & Safety Constraints

⚠️ **Important**:
- ❌ Never execute git commands automatically without confirmation
- ✅ Only suggest commands for manual execution (interactive mode)
- ✅ Wait for explicit user/pipeline confirmation (agent mode)
- ✅ Preserve important history information
- ✅ Maintain full change traceability
- ✅ No AI traces in commit messages

## Post-Commit Recommendations

After committing, suggest:
```bash
# Verify with
git log --oneline -3

# If using Makefile
make test

# Before pushing
git status
```

## Error Handling

### Common Issues

**Issue**: Files shouldn't be committed
```
Solution: Review .gitignore and fix before committing
```

**Issue**: Commit message doesn't follow convention
```
Solution: Follow type(scope): description format shown above
```

**Issue**: Unstaged changes exist
```
Solution: Review with `git diff` before deciding what to commit
```

## Examples

### Example 1: Simple Feature Commit
```
Files: src/feature.go, tests/feature_test.go
Type: feat (new feature)
Scope: module-name
Result: feat(module-name): add new feature
```

### Example 2: Bug Fix with Multiple Files
```
Files: cmd/main.go, pkg/util/helper.go
Type: fix (bug fix)
Scope: utility
Result: fix(utility): handle edge case in helper function
```

### Example 3: Refactoring
```
Files: internal/engine/core.go, internal/engine/processor.go
Type: refactor (restructuring)
Scope: engine
Result: refactor(engine): reorganize processor logic
```

---

