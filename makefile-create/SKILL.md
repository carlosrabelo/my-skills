---
name: makefile-create
description: Reference for creating a Makefile from scratch for Go and Python projects. Covers mandatory opening lines, self-documenting help via grep/awk, variables, standard targets, delegate-to-make/ pattern, and monorepo orchestration.
mode: agent
category: tooling
shared: true
---

# Makefile Guide

Standard structure and patterns for writing Makefiles in Go and Python projects.

## Overview

The Makefile is the single developer interface for a project — you always type `make <something>`, never a raw tool command directly. It should be short, self-documenting, and delegate complex logic to `make/` scripts.

**Key principle**: The Makefile orchestrates; `make/` scripts do the work. If a target needs more than two or three shell lines, extract it to a script.

Use this skill when creating a Makefile for the first time. To update an existing Makefile that deviates from the standard, use `makefile-migrate`.

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
.PHONY: build clean fmt help install lint quality run setup test typecheck uninstall version
```

List targets alphabetically. A target missing from `.PHONY` will silently do nothing if a file with that name exists in the directory.

❌ Do not scatter `.PHONY` declarations throughout the file — it makes it easy to miss entries.

---

## Variables Block

Define variables after `.PHONY`. Use only what the project actually needs.

**Go projects:**

```makefile
BINARY_NAME := project-name
BUILD_DIR   := bin
INSTALL_DIR := $(HOME)/.local/bin

VERSION    := $(shell git describe --tags --always --dirty 2>/dev/null || echo dev)
BUILD_TIME := $(shell date -u +%Y-%m-%dT%H:%M:%SZ)
LDFLAGS    := -s -w -X main.version=$(VERSION) -X main.buildTime=$(BUILD_TIME)
```

- `BINARY_NAME` — matches the repository name
- `BUILD_DIR` — always `bin`; never hard-code `bin/` inside targets
- `INSTALL_DIR` — `$(HOME)/.local/bin` for user installs; override with `sudo make install INSTALL_DIR=/usr/local/bin` for system installs
- `VERSION` — derived from `git describe`; falls back to `dev` when there are no tags or no git repo
- `LDFLAGS` — passed to `go build -ldflags` to embed version metadata

**Python projects** do not use LDFLAGS or VERSION variables in the Makefile (version lives in `pyproject.toml`). A Python Makefile typically has no variables block at all.

---

## Self-Documenting `help` Target

The `help` target reads `##` inline comments from each target and formats them into a usage table:

```makefile
help: ## Show available targets
	@echo "project-name - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'
```

Every target that should appear in the help output must have an `## inline comment`:

```makefile
build: ## Build binary with version metadata
test:  ## Run all tests
lint:  ## Run linter
```

Targets without `##` are silently excluded from the help output. Use this intentionally for internal helper targets that users do not need to know about.

---

## Standard Targets

### Build and Install

```makefile
build: ## Build binary
	@./make/build.sh

run: build ## Run the binary
	@./$(BUILD_DIR)/$(BINARY_NAME)

install: build ## Install binary to $(INSTALL_DIR)
	@./make/install.sh --user

uninstall: ## Remove installed binary
	@./make/uninstall.sh --user

clean: ## Remove build artifacts and caches
	@rm -rf $(BUILD_DIR)
	@go clean -cache -testcache 2>/dev/null || true
```

`run` is optional — include it only when the project has a meaningful "run" mode.

### Quality

```makefile
fmt: ## Format Go sources
	@go fmt ./...

vet: ## Run go vet
	@go vet ./...

lint: ## Run golangci-lint
	@golangci-lint run ./...

quality: fmt vet lint ## Run all quality checks
```

`quality` is a meta-target that groups all quality checks. Run it before committing. The order matters: format first, then vet, then lint.

For `lint`: if `golangci-lint` is not available, fall back to `go vet ./...` alone and document it in the comment.

### Test

```makefile
test: ## Run all tests
	@./make/test.sh
```

### Info

```makefile
version: ## Show version
	@echo "$(BINARY_NAME) $(VERSION) ($(BUILD_TIME))"
```

### Python-only

```makefile
setup: ## Create .venv and install dependencies
	@./make/setup.sh

fmt: ## Format Python sources with ruff
	@.venv/bin/ruff format .

lint: ## Lint Python sources with ruff
	@.venv/bin/ruff check .

typecheck: ## Type-check with mypy
	@.venv/bin/mypy .

quality: fmt lint typecheck ## Run all quality checks
```

`setup` is the first target a new contributor runs. It must create `.venv` and install all dependencies so the project is ready to use. See `python-project-create` for the `make/setup.sh` contents.

---

## Delegate-to-`make/` Pattern

If a target needs more than two or three shell lines, extract the logic to a `make/` script and delegate:

```makefile
build: ## Build binary
	@./make/build.sh

test: ## Run all tests
	@./make/test.sh
```

The `@` prefix suppresses echoing the command. Scripts get version information from environment variables or Makefile variables passed as arguments.

**What stays inline** (simple enough to keep in the Makefile):

```makefile
clean: ## Remove build artifacts
	@rm -rf bin/
	@go clean -cache -testcache 2>/dev/null || true

fmt: ## Format Go sources
	@go fmt ./...

version: ## Show version
	@echo "$(BINARY_NAME) $(VERSION) ($(BUILD_TIME))"
```

---

## Monorepo Orchestration

In a monorepo, the root Makefile is an orchestrator. It delegates to component scripts — it does not contain build logic itself.

**Pattern 1: direct script delegation**

```makefile
build: ## Build all components
	@go-component/make/build.sh
	@python-component/make/setup.sh

test: ## Test all components
	@go-component/make/test.sh
	@python-component/make/test.sh
```

**Pattern 2: `$(MAKE) -C` delegation (for symmetric components)**

```makefile
build: ## Build all components
	@$(MAKE) -C go-component build
	@$(MAKE) -C python-component build

test: ## Test all components
	@$(MAKE) -C go-component test
	@$(MAKE) -C python-component test
```

Each component has its own Makefile following this same guide. The root Makefile only orchestrates.

---

## Full Go Example

Complete Makefile for a typical Go CLI project:

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build clean fmt help install lint quality run test uninstall version vet

BINARY_NAME := myapp
BUILD_DIR   := bin
INSTALL_DIR := $(HOME)/.local/bin

VERSION    := $(shell git describe --tags --always --dirty 2>/dev/null || echo dev)
BUILD_TIME := $(shell date -u +%Y-%m-%dT%H:%M:%SZ)
LDFLAGS    := -s -w -X main.version=$(VERSION) -X main.buildTime=$(BUILD_TIME)

help: ## Show available targets
	@echo "$(BINARY_NAME) - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

build: ## Build binary
	@./make/build.sh

run: build ## Run the binary
	@./$(BUILD_DIR)/$(BINARY_NAME)

test: ## Run all tests
	@./make/test.sh

fmt: ## Format Go sources
	@go fmt ./...

vet: ## Run go vet
	@go vet ./...

lint: ## Run golangci-lint
	@golangci-lint run ./...

quality: fmt vet lint ## Run all quality checks

install: build ## Install binary to $(INSTALL_DIR)
	@./make/install.sh --user

uninstall: ## Remove installed binary
	@./make/uninstall.sh --user

clean: ## Remove build artifacts and caches
	@rm -rf $(BUILD_DIR)
	@go clean -cache -testcache 2>/dev/null || true

version: ## Show version
	@echo "$(BINARY_NAME) $(VERSION) ($(BUILD_TIME))"
```

---

## Full Python Example

Complete Makefile for a typical Python project:

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: clean fmt help install lint quality setup test typecheck uninstall

help: ## Show available targets
	@echo "myapp - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

setup: ## Create .venv and install dependencies
	@./make/setup.sh

test: ## Run all tests
	@./make/test.sh

fmt: ## Format sources with ruff
	@.venv/bin/ruff format .

lint: ## Lint sources with ruff
	@.venv/bin/ruff check .

typecheck: ## Type-check with mypy
	@.venv/bin/mypy .

quality: fmt lint typecheck ## Run all quality checks

install: ## Install entry points
	@./make/install.sh

uninstall: ## Remove installed entry points
	@./make/uninstall.sh

clean: ## Remove build artifacts and caches
	@rm -rf __pycache__ .mypy_cache .ruff_cache dist *.egg-info
	@find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null || true
```

---

## Anti-Patterns

❌ **Missing `MAKEFLAGS += --no-print-directory`** — directory enter/leave messages pollute output whenever you use `$(MAKE) -C`.

❌ **Missing `.DEFAULT_GOAL := help`** — bare `make` builds the first target instead of showing usage.

❌ **Targets without `##` inline comments** — the `help` target produces nothing useful; contributors have to read the Makefile to learn what targets exist.

❌ **Multi-line shell logic inside the Makefile** — extraction to `make/` scripts makes logic testable, readable, and reusable from CI without going through Make.

❌ **Two Makefiles for one language project** — a single root Makefile is the rule. The monorepo exception (root orchestrator + component Makefiles) does not apply to single-component projects.

❌ **Scattered `.PHONY` declarations** — put all non-file targets in one `.PHONY` line at the top so it is easy to audit.

❌ **Tabs replaced by spaces** — Make requires hard tab characters (`\t`) for recipe lines. Many editors silently replace tabs with spaces, causing cryptic `missing separator` errors. Configure your editor to preserve tabs in Makefiles.

---

## Key Principles

- Start every Makefile with `MAKEFLAGS += --no-print-directory` and `.DEFAULT_GOAL := help`
- Every user-visible target gets an `##` inline doc comment
- Delegate complex logic to `make/` scripts; keep the Makefile short
- Declare all non-file targets in a single `.PHONY` line at the top
- Use `quality: fmt vet lint` (Go) or `quality: fmt lint typecheck` (Python) as a commit-readiness shortcut

---

## Related Skills

- `go-project-create` — covers the `make/` scripts that Makefile targets delegate to
- `python-project-create` — covers the `setup` target, `.venv` conventions, and `make/setup.sh`
- `monorepo-project-create` — root orchestrator Makefile pattern for multi-component repos
