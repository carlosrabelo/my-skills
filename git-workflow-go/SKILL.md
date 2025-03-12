---
name: git-workflow-go
description: Guide to organizing Git commits and workflow for Go projects. Covers atomic commits, clear messages, respecting the src/ structure, refactoring vs feature changes, and maintaining clean history.
mode: manual
category: go
shared: true
---

# Git Workflow for Go Projects

Guide to organizing Git commits and maintaining clean history for Go projects that use the standard structure. Covers atomic commits, message conventions, refactoring strategies, and best practices for collaboration.

## Overview

Good Git workflow for Go projects:
- ✅ Commits are **atomic** (one logical change per commit)
- ✅ Commit messages are **clear and descriptive**
- ✅ Refactoring and features are in **separate commits**
- ✅ Build artifacts and generated files are **not committed**
- ✅ History is **clean and readable**

---

## Core Principles

### 1. Atomic Commits

**One logical change per commit.** A commit should be:
- Compilable (`go build` works)
- Testable (`go test` passes)
- Reversible (can be reverted without breaking things)
- Complete (includes code, tests, and doc updates)

**Good**:
```
Commit 1: Add package discovery for system detection
- Add src/internal/discovery/discover.go
- Add src/internal/discovery/types.go
- Add src/internal/discovery/discover_test.go
- Update README with usage example

Commit 2: Add error types for discovery package
- Add src/internal/discovery/errors.go
- Update discover.go to use new error types
- Update discover_test.go to test error cases
```

**Bad**:
```
Commit 1: Add discovery, validation, and output formatting
- Add 5 packages with dozens of functions
- Update documentation
- Add half of the tests
- (Can't test this commit in isolation)
```

### 2. Separation of Concerns

**Keep these in separate commits**:

| Type | Own Commit | Reason |
|------|-----------|--------|
| Feature | ✅ Yes | New functionality |
| Bug fix | ✅ Yes | Fixes existing issue |
| Refactoring | ✅ Yes | No functional change |
| Tests | ✅ Maybe | If adding new tests for feature |
| Documentation | ✅ Maybe | If not directly related to code |
| Formatting | ✅ Yes | If lots of changes |

**Example workflow**:
```
1. Fix bug in processor.go
   (1 commit - just the bug fix)

2. Refactor processor.go to be more readable
   (1 separate commit - pure refactoring, no functional change)

3. Add caching feature to processor
   (1 commit - new functionality)

4. Update README with cache usage examples
   (1 commit - documentation)
```

---

## Commit Messages

### Message Format

**First line (subject)**:
- Maximum 50 characters
- Imperative mood: "add feature" not "added feature" or "adds feature"
- Start with a verb when possible
- No period at the end

**Blank line**:
- Always include a blank line after subject

**Body (optional)**:
- Explain the **why**, not the **what**
- Wrap at 72 characters
- Use present tense
- Separate multiple paragraphs with blank lines

**Example format**:
```
Add discovery package for detecting available packages

This commit introduces the discovery package to detect installed
packages on the system. It provides an interface for discovering
packages across different package managers (apt, snap, flatpak).

The package is used by the main command to build an inventory
of what's currently installed, which is compared against other
systems to identify discrepancies.

Fixes #123
```

### Good Commit Message Examples

**Example 1: Feature**
```
Add package comparison and diff generation

Add the diff package that compares two system inventories
and generates a list of discrepancies. This is the core logic
used to identify which packages are missing or extra on each
system.

The diff is used by the suggestion package to generate
installation/removal commands for the user.
```

**Example 2: Bug Fix**
```
Fix nil pointer in processor when input is empty

The Process function would panic if given an empty input slice.
Add a check at the start of the function to handle empty input
gracefully and return an appropriate error.

Add test case for empty input.

Fixes #456
```

**Example 3: Refactoring**
```
Refactor discovery package into smaller files

Split src/internal/discovery/discover.go into focused files:
- discover.go (main public interface)
- detection.go (detection logic)
- helpers.go (utility functions)

No functional changes. All tests still pass.
```

**Example 4: Documentation**
```
Update README with discovery package examples

Add a section explaining how to use the discovery package,
including:
- Basic usage example
- Error handling patterns
- Supported package managers
```

### Bad Commit Message Examples

```
❌ "fix stuff"
❌ "WIP"
❌ "asdf"
❌ "Update code"
❌ "Added the discovery thing with some changes"
❌ "Fixed bugs and added features and refactored code"
```

---

## Commits in the src/ Structure

### Build Artifacts: Never Commit

```bash
# ❌ Never add to git
bin/myapp
src/coverage.out
src/coverage.html

# Add to .gitignore
echo "bin/" >> .gitignore
echo "src/coverage.*" >> .gitignore
```

### Source Code: Always Commit

```bash
# ✅ Always commit
src/go.mod
src/go.sum
src/cmd/
src/internal/
src/testdata/
src/*.go

# ✅ Root files
Makefile
run/
cfg/
README.md
LICENSE
```

### Commits Affecting Multiple Directories

When a change spans cmd/ and internal/:

```
Fix bug in command that uses internal package

The bug was in how the command invoked the processor package.
This commit includes:
- Fix in src/cmd/main/main.go
- Fix in src/internal/processor/processor.go
- Updated tests in both packages

The bug manifested as... (explain why, not what)
```

---

## Refactoring Commits

### How to Handle Refactoring

**Keep refactoring separate from feature changes:**

```bash
# Step 1: Commit refactoring (no functional changes)
git add src/internal/processor/
git commit -m "Refactor processor package into focused files

Split processor.go into:
- processor.go (public interface)
- loading.go (load logic)
- processing.go (core processing)
- formatting.go (output formatting)

No functional changes. All tests pass."

# Step 2: Make your feature change
git add src/internal/processor/
git commit -m "Add caching to processor

Cache results of expensive processing operations
using an LRU cache with 1-hour TTL.

This speeds up repeated processing of the same
input by approximately 10x."
```

### Why Separate Refactoring

- **Easier to review**: Reviewer can focus on logic in one commit, changes in the other
- **Easier to revert**: Can revert refactoring without losing features, or vice versa
- **Easier to debug**: `git bisect` will land on the exact commit that caused a problem
- **Git blame**: `git log -p` shows intent separately from changes

---

## Common Workflows

### Feature Development

```bash
# 1. Create feature branch
git checkout -b feature/add-caching

# 2. Work on feature
cd src && make test         # Make sure tests pass

# 3. Commit with clear message
git add src/
git commit -m "Add caching to processor

Cache expensive operations with LRU eviction.
Improves repeated processing by 10x."

# 4. Push and create PR
git push origin feature/add-caching
```

### Bug Fix

```bash
# 1. Create bug fix branch
git checkout -b fix/nil-pointer

# 2. Write test that reproduces the bug
cd src && go test ./... # Test fails

# 3. Fix the bug
# (edit code)

# 4. Verify test passes
cd src && go test ./... # Test passes

# 5. Commit
git add src/
git commit -m "Fix nil pointer panic in processor

Process would panic if given nil input. Add nil check
at the start of the function to return a clear error
instead."
```

### Refactoring Project

```bash
# 1. Create refactoring branch
git checkout -b refactor/split-files

# 2. Make refactoring changes
# (move code, update imports, no functional changes)

# 3. Verify everything still works
cd src && go test ./...
cd src && go build ./...

# 4. Commit as pure refactoring
git add src/
git commit -m "Refactor processor into focused files

Split processor.go into:
- processor.go (public interface)
- loading.go (file loading logic)
- processing.go (core processing algorithm)
- formatting.go (output formatting)

No functional changes. All tests pass. Code is
more maintainable and easier to test."
```

### Adding Tests

```bash
# 1. Add test for existing feature
cd src && go test -v ./...  # Tests pass

# 2. Add edge case tests
git add src/internal/processor/processor_test.go
git commit -m "Add tests for processor edge cases

Add tests for:
- Empty input handling
- Nil input handling
- Very large input
- Unicode input

All tests pass."
```

---

## .gitignore for Go Projects

Create `./gitignore` at project root:

```gitignore
# Build artifacts
bin/

# Go-specific
*.o
*.so
.DS_Store

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Test coverage
src/coverage.out
src/coverage.html

# Dependency downloads (optional, usually track go.mod/go.sum)
# vendor/

# OS files
Thumbs.db
```

---

## Commit Workflow for Teams

### Pull Request Template

Create `.github/pull_request_template.md`:

```markdown
## Description
Brief description of what this PR does.

## Type of change
- [ ] Bug fix
- [ ] New feature
- [ ] Refactoring
- [ ] Documentation

## Related issues
Fixes #123

## Testing
How did you test this change?
- [ ] Unit tests added/updated
- [ ] Manual testing
- [ ] Tested locally with: `make test && make build`

## Checklist
- [ ] Code follows style guidelines
- [ ] Comments added for complex logic
- [ ] Tests pass locally
- [ ] No new warnings or errors
- [ ] Commits are atomic and well-messaged
```

### Review Checklist for Reviewers

When reviewing commits:

- [ ] Commit message is clear and explains **why**
- [ ] Commit is atomic (one logical change)
- [ ] Tests pass
- [ ] Code builds
- [ ] Related changes are in separate commits
- [ ] No build artifacts committed
- [ ] Comments are clear and helpful
- [ ] No obvious bugs or issues

---

## Useful Git Commands for Go Projects

### Before committing

```bash
# Check what changed
git status
git diff                    # See exact changes
git diff --stat             # See file summary

# Verify code quality
cd src && go test ./...     # Tests pass
cd src && go build ./...    # Code compiles
cd src && go fmt ./...      # Format code
```

### Staging changes

```bash
# Add specific files
git add src/internal/processor/processor.go

# Add all changes in directory
git add src/internal/

# Interactive staging (choose which hunks to add)
git add -p
```

### Committing

```bash
# Commit with message
git commit -m "Clear commit message"

# Commit with editor for longer message
git commit              # Opens your editor

# Amend last commit (before pushing)
git commit --amend      # Edit last commit
```

### History

```bash
# View recent commits
git log --oneline -10

# View commits for a file
git log -p src/internal/processor/processor.go

# View commits for a directory
git log -p src/internal/processor/

# Search commits by message
git log --grep="caching"

# View who changed what
git blame src/internal/processor/processor.go
```

### Debugging with Git

```bash
# Find which commit introduced a bug
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# ... bisect will guide you

# Show what changed between commits
git diff commit1 commit2

# Show changes in a specific commit
git show commit-hash
```

---

## Commit Message Conventions Summary

| Element | Format | Example |
|---------|--------|---------|
| **Subject** | Imperative, ≤50 chars | "Add caching to processor" |
| **Tense** | Present | "Add" not "Added" |
| **Scope** | Can include in parens | "(processor) Add caching" |
| **Body** | Explain why | "Improves performance of..." |
| **Footer** | Issue reference | "Fixes #123" |

---

## Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: God Commits

```
❌ BAD: One huge commit with everything
Commit: "Major update"
- Add feature A
- Fix bug B
- Refactor package C
- Update documentation
- Add new tests
- Fix formatting

✅ GOOD: Separate logical commits
Commit 1: Add feature A
Commit 2: Fix bug B
Commit 3: Refactor package C
Commit 4: Update documentation for features
Commit 5: Add tests for new features
```

---

### ❌ Anti-Pattern 2: Mixing Refactoring and Features

```
❌ BAD: Refactoring and feature in same commit
Commit: "Add caching and refactor processor"
- Split processor.go into files
- Add caching logic
- Update imports
- Add tests

✅ GOOD: Separate commits
Commit 1: Refactor processor into focused files
(no functional change, all tests pass)

Commit 2: Add caching to processor
(new feature, builds on refactored structure)
```

---

### ❌ Anti-Pattern 3: Committing Build Artifacts

```
❌ BAD: Committing binaries and generated files
git add bin/myapp
git add src/coverage.out
git commit

✅ GOOD: Only commit source code
# Add to .gitignore
echo "bin/" >> .gitignore
echo "src/coverage.*" >> .gitignore
git add .gitignore
git commit
```

---

### ❌ Anti-Pattern 4: Vague Commit Messages

```
❌ BAD
- "fix stuff"
- "updates"
- "work in progress"
- "as per requirements"

✅ GOOD
- "Fix nil pointer panic in processor"
- "Add caching to improve performance"
- "Refactor validation into separate package"
- "Update README with usage examples"
```

---

### ❌ Anti-Pattern 5: Broken Commits

```
❌ BAD: Commit that doesn't compile/test
git add src/
git commit "Add new feature"
# But: go build fails, tests fail

✅ GOOD: Test before committing
cd src && go test ./...   # Pass
cd src && go build ./...  # Success
git add src/
git commit "Add new feature"
```

---

### ❌ Anti-Pattern 6: Incomplete Commits

```
❌ BAD: Feature without tests
Commit: "Add caching"
- Add caching logic
- (No tests for caching)

✅ GOOD: Feature with tests
Commit: "Add caching"
- Add caching logic
- Add caching_test.go with test cases
- Update processor_test.go with integration tests
```

---

## Example: Good vs Bad Commit History

### ❌ Bad History (Don't Do This)

```
ae3d4f1 Major refactor and feature additions
2c1a9e7 Fix stuff
5f2b8a3 WIP
9d4e1c2 more changes
7b3f5a1 Updates
```

You can't tell what each commit does, hard to review, hard to debug.

### ✅ Good History (Do This)

```
ae3d4f1 Update README with configuration examples
2c1a9e7 Add tests for validation edge cases
5f2b8a3 Refactor validation into separate package
9d4e1c2 Fix nil pointer panic in processor
7b3f5a1 Add caching to processor for performance
3c2e1a5 Add discovery package for package detection
```

Each commit is clear, atomic, and explains what was done and why.

---

## Checklist Before Pushing

```bash
# 1. Code quality
cd src && go test ./...     # ✅ Tests pass
cd src && go build ./...    # ✅ Code compiles
cd src && go fmt ./...      # ✅ Code formatted

# 2. Commits
git log -3 --oneline        # ✅ Clear, atomic commits
git diff origin/main        # ✅ Only intended changes

# 3. Repository state
git status                  # ✅ Clean working directory
git log --oneline -n 5      # ✅ Sensible commit history

# 4. Push
git push origin branch-name
```

---

## Related Skills

- **git-commit-suggest** — Tactical complement: analyzes current changes and suggests commit commands
- **go-project-structure** — The src/ structure that commits should respect and preserve
- **go-reorganize-refactor** — Refactoring changes become separate, dedicated commits
- **go-commenting-en** — Good code comments and clear commit messages go hand in hand

---

## Changelog

- **v1.0** (2025-03-12) — Initial version
  - Atomic commits and separation of concerns
  - Commit message conventions
  - Refactoring vs feature workflows
  - Team workflows and PR templates
  - Useful Git commands for Go projects
  - Anti-patterns to avoid
  - Pre-push checklist
