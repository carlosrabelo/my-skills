---
name: makefile-skeleton
description: Standard Makefile structure (opening lines, .PHONY, self-doc help, .make/ script delegation). Creates from scratch or updates existing files.
mode: agent
category: tooling
shared: true
---

# Makefile Skeleton

Unified skill for writing project Makefiles following a consistent pattern. Handles both new projects and updating existing files that deviate from the standard.

## Context Detection

Before starting, determine the context:

1. **Check for `Makefile`** in the current directory (or the target directory):
   ```bash
   find . -name "Makefile" -maxdepth 1
   ```

2. **If `Makefile` exists** → this is an existing file that needs updating. Follow **## Migrating an Existing Makefile** below.

3. **If `Makefile` does not exist** → create from scratch. Follow **## Creating from Scratch** below.

4. **In both cases**, the canonical format rules in **## Canonical Makefile Format** apply.

---

## Migrating an Existing Makefile

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the Makefile looks "mostly correct".**

### Mandatory Checklist

**Before:**
- [ ] Read the entire Makefile
- [ ] List all targets that exist
- [ ] Identify which standard targets are missing
- [ ] Note targets with multi-line shell (candidates for `.make/` extraction)
- [ ] Check `.make/` directory for existing scripts

**During:**
- [ ] Add mandatory opening lines (Scenario 1) first — this is always safe
- [ ] Consolidate `.PHONY` (Scenario 3) — no behavior change, safe to do early
- [ ] Convert help target (Scenario 2) — then verify `make help` output looks correct
- [ ] Add variables block (Scenario 6) — before updating targets that use them
- [ ] Extract shell to `.make/` scripts (Scenario 4) — test each target after extraction
- [ ] Add missing standard targets (Scenario 5) — test each new target

**After:**
- [ ] `make help` shows all expected targets
- [ ] `make build` produces the expected output
- [ ] `make test` runs and passes
- [ ] `make quality` runs without errors
- [ ] `make clean` removes build artifacts

### Migration Scenarios

#### Scenario 1: Add Mandatory Opening Lines

**When**: The Makefile does not start with `MAKEFLAGS` and/or `.DEFAULT_GOAL`.

**Before:**
```makefile
BINARY_NAME := myapp

.PHONY: build test
```

**After:**
```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build test
```

Steps:
1. Insert `MAKEFLAGS += --no-print-directory` as the very first line.
2. Leave a blank line.
3. Insert `.DEFAULT_GOAL := help` as the third line.
4. Leave a blank line before anything else.

If the file already has one but not the other, add only the missing line in the correct position.

---

#### Scenario 2: Convert Printf-Based Help to Grep/Awk

**When**: The `help` target uses `@printf` or `@echo` with hard-coded target names. Changing or adding a target requires manually updating the help text.

**Before (printf pattern):**
```makefile
help:
	@printf "\nBuild:\n"
	@printf "  build       Compile binary\n"
	@printf "  install     Install binary\n"
	@printf "\nQuality:\n"
	@printf "  lint        Run linter\n"
	@printf "  fmt         Format code\n"
```

**After (grep+awk pattern):**
```makefile
help: ## Show available targets
	@echo "myapp - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'
```

Steps:
1. Replace the entire `help` target body with the grep+awk block above.
2. Add `## description` inline comments to every target that should appear in help:
   ```makefile
   build: ## Build binary
   install: build ## Install binary to $(INSTALL_DIR)
   lint: ## Run linter
   fmt: ## Format code
   ```
3. Targets without `##` are silently excluded — use this for internal helper targets.
4. Add `help` to the `.PHONY` list if not already present.

---

#### Scenario 3: Consolidate `.PHONY`

**When**: `.PHONY` declarations are scattered through the file, or each target has its own `.PHONY` line.

**Before:**
```makefile
.PHONY: build
build:
	@./.make/build.sh

.PHONY: test
test:
	@./.make/test.sh

.PHONY: clean
clean:
	@rm -rf bin/
```

**After:**
```makefile
.PHONY: build clean help install lint test uninstall version
```

Steps:
1. Collect every non-file target name from all `.PHONY` lines.
2. Sort them alphabetically.
3. Replace all `.PHONY` lines with a single declaration at the top of the file, after the opening lines and before the variables block.
4. Remove the individual `.PHONY: target` lines scattered through the file.

---

#### Scenario 4: Extract Multi-Line Shell to `.make/`

**When**: A target contains more than 2-3 shell lines. The logic is hard to read, test, or reuse.

**Before:**
```makefile
build:
	@echo "Building $(BINARY_NAME)..."
	@mkdir -p bin
	@CGO_ENABLED=0 go build \
		-ldflags="$(LDFLAGS)" \
		-o bin/$(BINARY_NAME) \
		./cmd/$(BINARY_NAME)
	@echo "Done: bin/$(BINARY_NAME)"
```

**After:**
```makefile
build: ## Build binary
	@./.make/build.sh
```

And `.make/build.sh`:
```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
BINARY_NAME="myapp"
BUILD_DIR="$ROOT_DIR/bin"
VERSION=$(git describe --tags --always --dirty 2>/dev/null || echo dev)
BUILD_TIME=$(date -u +%Y-%m-%dT%H:%M:%SZ)
LDFLAGS="-s -w -X main.version=$VERSION -X main.buildTime=$BUILD_TIME"

mkdir -p "$BUILD_DIR"
echo "Building $BINARY_NAME..."
CGO_ENABLED=0 go build -ldflags="$LDFLAGS" -o "$BUILD_DIR/$BINARY_NAME" ./cmd/"$BINARY_NAME"
echo "Done: $BUILD_DIR/$BINARY_NAME"
```

Steps:
1. Create `.make/` directory if it does not exist.
2. Extract the shell lines to `.make/script-name.sh`.
3. Add `#!/usr/bin/env bash` and `set -euo pipefail` at the top of every new script.
4. Make the script executable: `chmod +x .make/script-name.sh`.
5. Replace the target body with `@./.make/script-name.sh`.
6. Variables that were in the Makefile (BINARY_NAME, VERSION, etc.) move into the script as local variables.

Common scripts to extract: `build.sh`, `test.sh`, `install.sh`, `uninstall.sh`, `setup.sh`.

---

#### Scenario 5: Add Missing Standard Targets

**When**: One or more standard targets are absent.

**`quality` meta-target** (groups all quality checks):
```makefile
quality: ## Run all quality checks
	@./.make/quality.sh
```

The composition of `quality` is language-specific — see the respective skeleton (go-skeleton, python-skeleton, etc.) for the exact dependencies (`fmt vet lint`, `fmt lint typecheck`, etc.).

Add the new target names to `.PHONY` after adding them.

---

#### Scenario 6: Add Variables Block

**When**: The Makefile hard-codes names, paths, or version strings directly inside targets.

Place the variables block after `.PHONY` and before the first target. Use `:=` (immediate assignment) for static values; use `$(shell ...)` for values derived at build time.

Language-specific variable patterns (Go's `BINARY_NAME`, `LDFLAGS`, `VERSION`; ESP32's `UPLOAD_PORT`) are defined in the respective skeleton. Refer to go-skeleton, python-skeleton, or esp32-skeleton for the canonical variables block for that language.

---

### Migration Rules

- Never remove an existing target name — add aliases or keep the old name as a dependency if renaming
- Keep backward-compatible target names so contributors with muscle memory are not broken
- Always test `make help` after converting the help target — if grep finds nothing, check that `##` comments are on the correct targets
- Scripts in `.make/` must be executable (`chmod +x`) before the Makefile delegates to them
- Do not add targets that are not needed by the project — migrate to the standard, not beyond it

---

## Creating from Scratch

The Makefile is the single developer interface for a project — you always type `make <something>`, never a raw tool command directly. It should be short, self-documenting, and delegate complex logic to `.make/` scripts.

**Key principle**: The Makefile orchestrates; `.make/` scripts do the work. If a target needs more than one shell line, extract it to a script.

**Why `.make/`**: Use a hidden dot-directory at the project root for shell scripts so the top-level tree stays focused on source and config. When refactoring older repos, rename `make/` → `.make/` and update Makefile paths accordingly.

Generic targets that apply to any project type:

```makefile
build: ## Build project
	@./.make/build.sh

test: ## Run tests
	@./.make/test.sh

clean: ## Remove build artifacts
	@./.make/clean.sh

quality: ## Run all quality checks
	@./.make/quality.sh
```

`quality` is a meta-target grouping all quality checks (format, lint, type-check, vet — varies by language). See the language skeleton for the specific composition.

Language-specific targets (`fmt`, `lint`, `vet`, `typecheck`, `setup`, `install`, `version`, `flash`, `monitor`, etc.) are defined in each respective skeleton.

**Complete generic example** (project without a specific language skeleton):

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build clean help quality test

help: ## Show available targets
	@echo "myproject - Available targets"
	@echo ""
	@grep -hE '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

build: ## Build project
	@./.make/build.sh

test: ## Run tests
	@./.make/test.sh

quality: ## Run all quality checks
	@./.make/quality.sh

clean: ## Remove build artifacts
	@./.make/clean.sh
```

---

## Canonical Makefile Format

### Mandatory Opening Lines

Every Makefile starts with these two lines, in this order:

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help
```

`MAKEFLAGS += --no-print-directory` suppresses the "Entering/Leaving directory" noise that appears when delegating with `$(MAKE) -C`. `.DEFAULT_GOAL := help` ensures that bare `make` (with no target) shows usage instead of building something unexpectedly.

These two lines are non-negotiable. They go before everything else.

### `.PHONY` Declaration

Declare all non-file targets in a single `.PHONY` line at the top, right after the opening lines:

```makefile
.PHONY: build clean help quality test
```

List targets alphabetically. A target missing from `.PHONY` will silently do nothing if a file with that name exists in the directory.

Do not scatter `.PHONY` declarations throughout the file.

### Variables Block

Define variables after `.PHONY`. Use only what the project actually needs.

Language-specific variables (Go's `BINARY_NAME`, `LDFLAGS`, `VERSION`; ESP32's `UPLOAD_PORT`) are defined in the respective skeleton. A generic project with no build metadata typically has no variables block.

### Self-Documenting `help` Target

The `help` target reads `##` inline comments from each target and formats them into a usage table:

```makefile
help: ## Show available targets
	@echo "project-name - Available targets"
	@echo ""
	@grep -hE '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'
```

Every target that should appear in the help output must have an `##` inline comment. Targets without `##` are silently excluded — use this for internal helper targets.

### Delegate-to-`.make/` Pattern

If a target needs more than one shell line, extract the logic to a `.make/` script and delegate:

```makefile
build: ## Build project
	@./.make/build.sh

test: ## Run tests
	@./.make/test.sh
```

The `@` prefix suppresses echoing the command. Scripts start with `#!/usr/bin/env bash` and `set -euo pipefail`.

**What stays inline** (single-line targets):

```makefile
fmt: ## Format sources
	@tool format .

version: ## Show version
	@echo "$(VERSION)"
```

### Anti-Patterns

- **`all` target** — ambiguous; use named targets (`build`, `test`, `quality`).
- **Missing `MAKEFLAGS += --no-print-directory`** — directory messages pollute output with `$(MAKE) -C`.
- **Missing `.DEFAULT_GOAL := help`** — bare `make` builds the first target instead of showing usage.
- **Targets without `##` inline comments** — `help` produces nothing useful.
- **More than one shell line inside a target** — extract to a `.make/` script.
- **Two Makefiles for one language project** — monorepo exception only.
- **Scattered `.PHONY` declarations** — one declaration at the top.
- **Tabs replaced by spaces** — Make requires hard tab characters (`\t`) for recipe lines.

---

## Monorepo Usage

In a monorepo, the root Makefile is an orchestrator. It delegates to component scripts — it does not contain build logic itself. Component Makefiles each follow the same standard structure.

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

---

## Related Skills

- **go-skeleton** — Complete Go Makefile with `BINARY_NAME`, `LDFLAGS`, `VERSION`, and `.make/` scripts
- **python-skeleton** — Complete Python Makefile with `setup`, `typecheck`, `.venv`, and `.make/setup.sh`
- **esp32-skeleton** — Complete ESP32 Makefile with `flash`, `monitor`, `install-pio`, and PlatformIO scripts
- **monorepo-skeleton** — Root orchestrator Makefile pattern for multi-component repos
- **project-scaffold** — Auto-detects project type and applies the appropriate skeleton
