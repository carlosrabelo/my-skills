# Makefile Guide

Standard structure and patterns for writing Makefiles. Language-specific targets (Go, Python, ESP32) and variables are defined in the respective skeleton skills.

## Overview

The Makefile is the single developer interface for a project — you always type `make <something>`, never a raw tool command directly. It should be short, self-documenting, and delegate complex logic to `make/` scripts.

**Key principle**: The Makefile orchestrates; `make/` scripts do the work. If a target needs more than one shell line, extract it to a script.

---

## Mandatory Opening Lines

Every Makefile starts with these two lines, in this order:

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help
```

`MAKEFLAGS += --no-print-directory` suppresses the "Entering/Leaving directory" noise that appears when delegating with `$(MAKE) -C`. `.DEFAULT_GOAL := help` ensures that bare `make` (with no target) shows usage instead of building something unexpectedly.

These two lines are non-negotiable. They go before everything else.

---

## `.PHONY` Declaration

Declare all non-file targets in a single `.PHONY` line at the top, right after the opening lines:

```makefile
.PHONY: build clean help quality test
```

List targets alphabetically. A target missing from `.PHONY` will silently do nothing if a file with that name exists in the directory.

❌ Do not scatter `.PHONY` declarations throughout the file.

---

## Variables Block

Define variables after `.PHONY`. Use only what the project actually needs.

Language-specific variables (Go's `BINARY_NAME`, `LDFLAGS`, `VERSION`; ESP32's `UPLOAD_PORT`) are defined in the respective skeleton. A generic project with no build metadata typically has no variables block.

---

## Self-Documenting `help` Target

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

---

## Standard Targets

Generic targets that apply to any project type:

```makefile
build: ## Build project
	@./make/build.sh

test: ## Run tests
	@./make/test.sh

clean: ## Remove build artifacts
	@./make/clean.sh

quality: ## Run all quality checks
	@./make/quality.sh
```

`quality` is a meta-target grouping all quality checks (format, lint, type-check, vet — varies by language). See the language skeleton for the specific composition.

Language-specific targets (`fmt`, `lint`, `vet`, `typecheck`, `setup`, `install`, `version`, `flash`, `monitor`, etc.) are defined in each respective skeleton.

---

## Delegate-to-`make/` Pattern

If a target needs more than one shell line, extract the logic to a `make/` script and delegate:

```makefile
build: ## Build project
	@./make/build.sh

test: ## Run tests
	@./make/test.sh
```

The `@` prefix suppresses echoing the command. Scripts start with `#!/usr/bin/env bash` and `set -euo pipefail`.

**What stays inline** (single-line targets):

```makefile
fmt: ## Format sources
	@tool format .

version: ## Show version
	@echo "$(VERSION)"
```

---

## Monorepo Orchestration

In a monorepo, the root Makefile is an orchestrator. It delegates to component scripts — it does not contain build logic itself. See **monorepo-skeleton** for full root Makefile patterns and component conventions.

---

## Generic Example

Complete Makefile for a project without a specific language skeleton:

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
	@./make/build.sh

test: ## Run tests
	@./make/test.sh

quality: ## Run all quality checks
	@./make/quality.sh

clean: ## Remove build artifacts
	@./make/clean.sh
```

---

## Anti-Patterns

❌ **`all` target** — ambiguous; use named targets (`build`, `test`, `quality`).

❌ **Missing `MAKEFLAGS += --no-print-directory`** — directory messages pollute output with `$(MAKE) -C`.

❌ **Missing `.DEFAULT_GOAL := help`** — bare `make` builds the first target instead of showing usage.

❌ **Targets without `##` inline comments** — `help` produces nothing useful.

❌ **More than one shell line inside a target** — extract to a `make/` script.

❌ **Two Makefiles for one language project** — monorepo exception only.

❌ **Scattered `.PHONY` declarations** — one declaration at the top.

❌ **Tabs replaced by spaces** — Make requires hard tab characters (`\t`) for recipe lines.

---

## Key Principles

- Start every Makefile with `MAKEFLAGS += --no-print-directory` and `.DEFAULT_GOAL := help`
- Every user-visible target gets an `##` inline doc comment
- Any target with more than one shell line must be extracted to a `make/` script
- Declare all non-file targets in a single `.PHONY` line at the top
- Language-specific targets and variables belong in the language skeleton, not here
