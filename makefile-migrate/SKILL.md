---
name: makefile-migrate
description: Reorganize existing Makefiles to match the standard structure. Covers adding mandatory opening lines, converting printf help to grep/awk, extracting shell logic to make/ scripts, adding .PHONY, and adding missing standard targets.
mode: manual
category: tooling
shared: true
---

# Makefile Migration

Step-by-step guide for bringing an existing Makefile up to the standard defined in `makefile-create`.

## Overview

Use this skill when a project has an existing Makefile that deviates from the standard — missing opening lines, a printf-based help target, multi-line shell logic, scattered `.PHONY`, or missing standard targets.

**Target state**: a Makefile that fully matches `makefile-create`:
- Opens with `MAKEFLAGS += --no-print-directory` and `.DEFAULT_GOAL := help`
- Has a single `.PHONY` declaration at the top
- Uses the grep+awk self-documenting help pattern
- Delegates complex logic to `make/` scripts
- Has all standard targets (`build`, `test`, `lint`, `fmt`, `clean`, `install`, `uninstall`, `quality`, `version`)

---

## Before You Start

1. Read the current Makefile in full — understand every target before changing anything.
2. Compare against `makefile-create` — note which standard elements are missing.
3. Check the `make/` directory — if scripts already exist, the delegation pattern may already be partially in place.
4. Identify targets with multi-line shell logic that should move to `make/` scripts.
5. Confirm that any targets you rename or add do not break CI workflows or documented commands.

---

## Migration Scenarios

### Scenario 1: Add Mandatory Opening Lines

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

### Scenario 2: Convert Printf-Based Help to Grep/Awk

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

### Scenario 3: Consolidate `.PHONY`

**When**: `.PHONY` declarations are scattered through the file, or each target has its own `.PHONY` line.

**Before:**
```makefile
.PHONY: build
build:
	@./make/build.sh

.PHONY: test
test:
	@./make/test.sh

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

### Scenario 4: Extract Multi-Line Shell to `make/`

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
	@./make/build.sh
```

And `make/build.sh`:
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
1. Create `make/` directory if it does not exist.
2. Extract the shell lines to `make/script-name.sh`.
3. Add `#!/usr/bin/env bash` and `set -euo pipefail` at the top of every new script.
4. Make the script executable: `chmod +x make/script-name.sh`.
5. Replace the target body with `@./make/script-name.sh`.
6. Variables that were in the Makefile (BINARY_NAME, VERSION, etc.) move into the script as local variables.

Common scripts to extract: `build.sh`, `test.sh`, `install.sh`, `uninstall.sh`, `setup.sh`.

---

### Scenario 5: Add Missing Standard Targets

**When**: One or more standard targets are absent.

**`quality` meta-target** (groups all quality checks):
```makefile
quality: fmt vet lint ## Run all quality checks
```
For Python: `quality: fmt lint typecheck`

**`version` target** (shows build metadata):
```makefile
version: ## Show version
	@echo "$(BINARY_NAME) $(VERSION) ($(BUILD_TIME))"
```
Requires the variables block (see Scenario 6).

**`setup` target** (Python only — creates `.venv` and installs deps):
```makefile
setup: ## Create .venv and install dependencies
	@./make/setup.sh
```

**`all` convenience target** (optional — runs the most common sequence):
```makefile
all: quality test build ## Run quality checks, tests, and build
```

Add the new target names to `.PHONY` after adding them.

---

### Scenario 6: Add Variables Block

**When**: The Makefile hard-codes binary names, paths, or version strings directly inside targets instead of using variables.

**Before:**
```makefile
build:
	@go build -ldflags="-s -w" -o bin/myapp ./cmd/myapp

install:
	@cp bin/myapp $(HOME)/.local/bin/myapp
```

**After:**
```makefile
BINARY_NAME := myapp
BUILD_DIR   := bin
INSTALL_DIR := $(HOME)/.local/bin

VERSION    := $(shell git describe --tags --always --dirty 2>/dev/null || echo dev)
BUILD_TIME := $(shell date -u +%Y-%m-%dT%H:%M:%SZ)
LDFLAGS    := -s -w -X main.version=$(VERSION) -X main.buildTime=$(BUILD_TIME)
```

Then update targets to use the variables:
```makefile
build: ## Build binary
	@go build -ldflags="$(LDFLAGS)" -o $(BUILD_DIR)/$(BINARY_NAME) ./cmd/$(BINARY_NAME)

install: build ## Install binary to $(INSTALL_DIR)
	@cp $(BUILD_DIR)/$(BINARY_NAME) $(INSTALL_DIR)/$(BINARY_NAME)
```

Place the variables block after `.PHONY` and before the first target. Use `:=` (immediate assignment) for static values; use `$(shell ...)` for values derived at build time.

---

## Migration Checklist

**Before:**
- [ ] Read the entire Makefile
- [ ] List all targets that exist
- [ ] Identify which standard targets are missing
- [ ] Note targets with multi-line shell (candidates for `make/` extraction)
- [ ] Check `make/` directory for existing scripts

**During:**
- [ ] Add mandatory opening lines (Scenario 1) first — this is always safe
- [ ] Consolidate `.PHONY` (Scenario 3) — no behavior change, safe to do early
- [ ] Convert help target (Scenario 2) — then verify `make help` output looks correct
- [ ] Add variables block (Scenario 6) — before updating targets that use them
- [ ] Extract shell to `make/` scripts (Scenario 4) — test each target after extraction
- [ ] Add missing standard targets (Scenario 5) — test each new target

**After:**
- [ ] `make help` shows all expected targets
- [ ] `make build` produces the binary in `bin/`
- [ ] `make test` runs and passes
- [ ] `make quality` runs lint, fmt, vet without errors
- [ ] `make version` shows version and build time
- [ ] `make clean` removes build artifacts
- [ ] `make install` installs the binary to the expected location
- [ ] `make uninstall` removes the installed binary

---

## Rules

- Never remove an existing target name — add aliases or keep the old name as a dependency if renaming
- Keep backward-compatible target names so contributors with muscle memory are not broken
- Always test `make help` after converting the help target — if grep finds nothing, check that `##` comments are on the correct targets
- Scripts in `make/` must be executable (`chmod +x`) before the Makefile delegates to them
- Do not add targets that are not needed by the project — migrate to the standard, not beyond it

---

---

## Related Skills

- `makefile-create` — the target state this skill migrates toward
- `go-project-create` — covers the `make/` scripts that Makefile targets delegate to
- `python-project-create` — covers `setup` target and `.venv` conventions
- `go-project-migrate` — invokes this skill as part of its Chaining sequence
- `python-project-migrate` — invokes this skill as part of its Chaining sequence
