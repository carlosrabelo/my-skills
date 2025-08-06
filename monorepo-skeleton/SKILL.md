---
name: monorepo-skeleton
description: Standard multi-language monorepo structure (component-level roots, root orchestrator Makefile, consistent naming). Creates from scratch or reorganizes existing repos.
mode: agent
category: project
shared: true
---

# Monorepo Skeleton

Unified skill for organizing multi-language monorepos following a consistent pattern. Handles both new monorepos and reorganization of existing repos.

## Context Detection

Before starting, determine the context:

1. **Check for language roots at the git root**:
   ```bash
   ls go.mod pyproject.toml package.json 2>/dev/null
   ```

2. **If language files exist at the git root** → this repo needs reorganization. Follow ## Migrating an Existing Repo to Monorepo Layout below.

3. **If no language files exist at the git root** (or the directory is empty/new) → create the monorepo structure from scratch. Follow ## Creating a New Monorepo from Scratch below.

4. **In both cases**, the canonical layout and anti-patterns in ## Canonical Layout and ## Anti-Patterns always apply.

---

## Migrating an Existing Repo to Monorepo Layout

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the repo looks "mostly correct".**

### Mandatory Checklist

#### Before
- [ ] `git status` is clean
- [ ] All tests pass
- [ ] Component names decided (domain-descriptive, not `backend/`, `src/`)

#### During
- [ ] Move files to `<component>/`
- [ ] Never move `.venv/` — recreate it inside the component
- [ ] Fix ROOT_DIR only if scripts moved to a different depth
- [ ] Root Makefile has no language-specific build logic — global targets (deploy, ci, release) are allowed

#### After
- [ ] `go test ./...` passes inside each Go component
- [ ] `make test` passes inside each Python component
- [ ] `make test` passes from git root
- [ ] No `go.mod` at git root
- [ ] No `pyproject.toml` or `.venv` at git root
- [ ] No `bin/` at git root
- [ ] Root Makefile has no language-specific logic
- [ ] Commit with clear message

### Diagnose the Repo

```bash
# What's at the root?
ls -la

# Are there language roots at the git root?
ls go.mod pyproject.toml 2>/dev/null

# What components already exist?
ls -d */ 2>/dev/null

# Where are existing .make/ scripts?
find . -name "*.sh" -not -path "./.venv/*" -not -path "*/vendor/*"
```

### Migration Scenarios

#### Scenario 1: Convert Single Go Repo to Monorepo Component

**Symptom**: A Go repo has `go.mod` at the git root and needs to become one component in a multi-language monorepo.

**Before**:
```
repo/
├── Makefile
├── go.mod
├── go.sum
├── bin/
├── .make/
│   ├── build.sh
│   └── test.sh
├── cmd/
└── internal/
```

**Steps**:

1. **Create the component directory**:
```bash
mkdir go-component    # use a domain-specific name
```

2. **Move all Go content into the component**:
```bash
mv go.mod go.sum go-component/
mv cmd/ internal/ go-component/
mv Makefile go-component/
mv bin/ go-component/
mv .make/ go-component/
mv .gitignore go-component/    # if it's Go-specific
```

3. **Fix ROOT_DIR in .make/ scripts** — was resolving to git root, now must resolve to `<component>/`:
```bash
# Before (resolved to repo/):
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

# After (resolves to go-component/):
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
# No change needed! The script is now at go-component/.make/,
# so $(dirname "$0")/.. still resolves to go-component/.
```

4. **Create the root orchestrator Makefile**:
```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build help test

help: ## Show available targets
	@echo "monorepo - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

build: ## Build all components
	@go-component/.make/build.sh

test: ## Test all components
	@go-component/.make/test.sh
```

5. **Create a root `.gitignore`** (if needed):
```
# Root .gitignore — only repo-level ignores
.DS_Store
```

6. **Verify**:
```bash
cd go-component && go test ./...
make build    # from repo root
make test
```

#### Scenario 2: Convert Single Python Repo to Monorepo Component

**Symptom**: A Python repo has `pyproject.toml` at the git root and needs to become one component.

**Before**:
```
repo/
├── Makefile
├── pyproject.toml
├── .venv/
├── .make/
│   ├── setup.sh
│   └── test.sh
├── main.py
└── processor.py
```

**Steps**:

1. **Deactivate any active venv** first:
```bash
deactivate 2>/dev/null || true
```

2. **Create the component directory**:
```bash
mkdir python-component    # use a domain-specific name
```

3. **Move all Python content into the component**:
```bash
mv pyproject.toml python-component/
mv Makefile python-component/
mv .make/ python-component/
mv tests/ python-component/    # if exists
mv *.py python-component/
# Move sub-packages if any:
# mv models/ python-component/
```

4. **Recreate `.venv` inside the component** (do not move it — paths get corrupted):
```bash
rm -rf .venv
cd python-component
python3 -m venv .venv
.make/setup.sh    # reinstalls dependencies
```

5. **Verify shebag paths** — `Path(__file__).resolve().parent` now resolves to `python-component/`, where `.venv` is a sibling. No changes needed if shebag used `parent` (not `parent.parent`).

6. **Create the root orchestrator Makefile**:
```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: help setup test

help: ## Show available targets
	@echo "monorepo - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

setup: ## Set up all components
	@python-component/.make/setup.sh

test: ## Test all components
	@python-component/.make/test.sh
```

7. **Verify**:
```bash
cd python-component && make test
make test    # from repo root
```

#### Scenario 3: Fix Go.mod at Git Root (Multi-Component Repo)

**Symptom**: A multi-component repo has `go.mod` at the git root — one language has "colonized" the repo root.

**Before**:
```
repo/
├── Makefile          ← mixes Go + Python targets
├── go.mod            ← wrong: at git root
├── go.sum
├── cmd/
├── internal/
└── python-component/ ← already isolated
    └── pyproject.toml
```

**Steps**:

1. **Create the Go component directory**:
```bash
mkdir api-server    # domain-specific name
```

2. **Move Go content**:
```bash
mv go.mod go.sum api-server/
mv cmd/ internal/ api-server/
mv bin/ .make/ api-server/    # if they exist and are Go-specific
```

3. **Split the Makefile**:
   - Create `api-server/Makefile` with Go-specific targets (build, test, lint, fmt)
   - Replace root `Makefile` with orchestrator that delegates to both components

4. **Fix Go import paths** — if any imports referenced relative paths, verify they still resolve. Usually no changes needed since `go.mod` module path is unchanged.

5. **Verify**:
```bash
cd api-server && go test ./...
make build && make test    # from root
```

#### Scenario 4: Merge Two Separate Repos into a New Monorepo

**Symptom**: Two repos (`go-service` and `python-pipeline`) that should be developed together.

**Steps**:

1. **Create the monorepo**:
```bash
mkdir monorepo && cd monorepo
git init
```

2. **Add each repo as a remote and merge its history**:
```bash
git remote add go-service /path/to/go-service
git fetch go-service

git remote add python-pipeline /path/to/python-pipeline
git fetch python-pipeline
```

3. **Create component directories and import each repo's content**:
```bash
# For go-service:
mkdir go-service
git read-tree --prefix=go-service/ -u go-service/main
git commit -m "import go-service as monorepo component"

# For python-pipeline:
mkdir python-pipeline
git read-tree --prefix=python-pipeline/ -u python-pipeline/main
git commit -m "import python-pipeline as monorepo component"
```

4. **Fix ROOT_DIR in .make/ scripts** — each component's scripts now live one level deeper (inside the component dir), but since they use `$(dirname "$0")/..`, they already resolve correctly.

5. **Create the root orchestrator Makefile**.

6. **Verify both components independently**, then from root.

#### Scenario 5: Rename a Component Directory

**Symptom**: A component has a generic name (`backend/`, `service/`, `app/`) that should be domain-descriptive.

**Steps**:

1. **Rename the directory**:
```bash
git mv backend/ api-server/
```

2. **Update the root Makefile** — any references to the old path:
```makefile
# Old:
build:
	backend/.make/build.sh

# New:
build:
	api-server/.make/build.sh
```

3. **Update CI/CD configs** — `.github/workflows/`, `Dockerfile`, etc.

4. **Verify**:
```bash
make build && make test
```

---

## Creating a New Monorepo from Scratch

The git root is an **orchestrator only**. No language "lives" at the git root. Each component is self-contained with its own root (`go.mod` or `pyproject.toml`), its own Makefile, and its own `.make/` scripts.

### Root Layout

```
monorepo/
├── Makefile              ← Orchestrator only — delegates to component .make/ scripts
├── go-component/         ← Go root (contains go.mod)
│   ├── Makefile
│   ├── go.mod
│   ├── bin/
│   └── .make/
├── python-component/     ← Python root (contains pyproject.toml)
│   ├── Makefile
│   ├── pyproject.toml
│   ├── .venv/
│   └── .make/
└── shared/               ← Optional: CI configs, shared scripts, docs
```

**Never** at the git root:
- `go.mod` or `go.sum`
- `pyproject.toml` or `.venv`
- `bin/` (each component has its own)

### Root Orchestrator Makefile

The root Makefile primarily delegates to components. It may also contain global-only targets — `deploy`, `release`, `ci`, `infra-apply`, `docs` — that span multiple components or have no single component owner. What it must never contain is language-specific build or test logic (`go build`, `pip install` directly in targets):

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build help test

help: ## Show available targets
	@echo "monorepo - Available targets"
	@echo ""
	@grep -E '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

build: ## Build all components
	@go-component/.make/build.sh
	@python-component/.make/setup.sh

test: ## Test all components
	@go-component/.make/test.sh
	@python-component/.make/test.sh
```

Having two Makefiles (root + component) is **not** the Anti-Pattern (Two Makefiles) in a multi-language monorepo — they serve different purposes. The anti-pattern is two Makefiles for the same language/component.

### When to Use a Monorepo

**Use a monorepo when**:
- Components share a deploy cycle and are developed together
- Teams are small enough to work across all components
- CI/CD benefits from unified pipeline orchestration

**Use separate repos when**:
- Projects are completely independent with different teams
- Release cycles are entirely decoupled
- Sharing code between them is rare or via published packages

---

## Canonical Layout

- The git root is an orchestrator only — no language roots there
- Each component is fully self-contained under its own subdirectory
- Components are never placed directly at the git root
- Use domain-descriptive names for component directories

### Component Naming

Use domain-descriptive names:

| Good | Bad |
|---|---|
| `api-server/` | `backend/` |
| `data-pipeline/` | `service1/` |
| `cli-tool/` | `src/` |
| `auth-go/`, `auth-py/` (only when names collide) | `lib/` |

Use language suffixes (`-go`, `-py`) only when two components share the same domain name.

### Component Self-Containment Rules

Each component:
- Has its own `Makefile` with language-specific targets
- Has its own `.make/` with scripts
- Resolves `ROOT_DIR` relative to itself, not the git root:
  ```bash
  ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"   # resolves to <component>/
  ```
- If a script genuinely needs the git root (e.g. shared output path):
  ```bash
  GIT_ROOT="$(cd "$(dirname "$0")/../.." && pwd)"
  ```

### Per-Language Components

- **Go components**: see `go-skeleton` — `go.mod` lives at `<component>/`, `bin/` and `.make/` are siblings
- **C++ components**: see `cpp-skeleton` — `projectname/` holds `src/` and `tests/`; prefer per-component `bin/` when multiple C++ trees exist
- **Python components**: see `python-skeleton` — `pyproject.toml` and `.venv` live at `<component>/`, self-relaunching shebag works unchanged

---

## Anti-Patterns

| Anti-Pattern | Why It's Wrong |
|---|---|
| `go.mod` at git root | Makes the monorepo a pure Go project; other components become sub-packages |
| `pyproject.toml` or `.venv` at git root | Python tooling takes over the entire repo |
| Root Makefile with language-specific build logic | Mixes orchestration with implementation — use component Makefiles for `go build`, `g++` / component scripts, `pip install`, etc.; global targets like `deploy` and `ci` are fine at root |
| Shared `bin/` at git root | Binary naming conflicts across components |
| Component named `src/` or `lib/` | No domain meaning; misleads future contributors |

---

## Monorepo Rules

- **Never move `.venv/`** — symlinks inside it are absolute; always recreate it in the new location
- **Never add language files to the git root** — not even temporarily
- **Rename directories with `git mv`**, not `mv`, to preserve history
- **Reorganize before adding new features** — separate commits for structure vs. behavior
- **Root Makefile must not contain language-specific build or test logic** — if you catch yourself writing `go build` or `pip install` directly in a root target, move it to the component Makefile. Global targets like `deploy`, `ci`, `release`, and `infra-apply` are fine at root.

---

## Chaining

After completing all steps above:

1. **For each Go component** — invoke the `go-skeleton` skill to verify and fix the internal structure
2. **For each C++ component** — invoke the `cpp-skeleton` skill to verify and fix the internal structure
3. **For each Python component** — invoke the `python-skeleton` skill to verify and fix the internal structure
4. **Check the root Makefile** — if it does not follow the `makefile-skeleton` standard, invoke the `makefile-skeleton` skill
5. **Check the root READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill
6. **Commit the changes** — invoke the `git-commit-suggest` skill to stage and commit the reorganization

## Related Skills

- **go-skeleton** — Internal structure of Go components (create from scratch or migrate existing)
- **cpp-skeleton** — Internal structure of C++ components (Makefile + `.make/` layout)
- **python-skeleton** — Internal structure of Python components (create from scratch or migrate existing)
- **makefile-skeleton** — Standard Makefile structure used in both root orchestrator and component Makefiles
