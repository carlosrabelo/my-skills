# Monorepo Migration

Step-by-step guide for reorganizing an existing repository into the standard monorepo structure defined in `references/create.md`.

## Overview

This skill handles three situations:

1. A single-language repo that needs to become one component of a larger monorepo
2. An existing monorepo where language roots are at the git root (wrong layout)
3. Merging two or more separate repos into a new monorepo

Target structure:
```
monorepo/
├── Makefile              ← Orchestrator only
├── go-component/         ← Go root (go.mod here)
│   ├── Makefile
│   ├── go.mod
│   ├── bin/
│   └── make/
└── python-component/     ← Python root (pyproject.toml here)
    ├── Makefile
    ├── pyproject.toml
    ├── .venv/
    └── make/
```

---

## Before You Start

### Checklist

- ✅ All changes committed (`git status` clean)
- ✅ All tests passing
- ✅ Understand what each component does
- ✅ Decide on component names (see Component Naming in `references/create.md`)

### Diagnose the Repo

```bash
# What's at the root?
ls -la

# Are there language roots at the git root?
ls go.mod pyproject.toml 2>/dev/null

# What components already exist?
ls -d */ 2>/dev/null

# Where are existing make/ scripts?
find . -name "*.sh" -not -path "./.venv/*" -not -path "*/vendor/*"
```

---

## Reorganization Scenarios

### Scenario 1: Convert Single Go Repo to Monorepo Component

**Symptom**: A Go repo has `go.mod` at the git root and needs to become one component in a multi-language monorepo.

**Before**:
```
repo/
├── Makefile
├── go.mod
├── go.sum
├── bin/
├── make/
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
mv make/ go-component/
mv .gitignore go-component/    # if it's Go-specific
```

3. **Fix ROOT_DIR in make/ scripts** — was resolving to git root, now must resolve to `<component>/`:
```bash
# Before (resolved to repo/):
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

# After (resolves to go-component/):
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
# No change needed! The script is now at go-component/make/,
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
	@go-component/make/build.sh

test: ## Test all components
	@go-component/make/test.sh
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

---

### Scenario 2: Convert Single Python Repo to Monorepo Component

**Symptom**: A Python repo has `pyproject.toml` at the git root and needs to become one component.

**Before**:
```
repo/
├── Makefile
├── pyproject.toml
├── .venv/
├── make/
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
mv make/ python-component/
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
make/setup.sh    # reinstalls dependencies
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
	@python-component/make/setup.sh

test: ## Test all components
	@python-component/make/test.sh
```

7. **Verify**:
```bash
cd python-component && make test
make test    # from repo root
```

---

### Scenario 3: Fix Go.mod at Git Root (Multi-Component Repo)

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
mv bin/ make/ api-server/    # if they exist and are Go-specific
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

---

### Scenario 4: Merge Two Separate Repos into a New Monorepo

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

4. **Fix ROOT_DIR in make/ scripts** — each component's scripts now live one level deeper (inside the component dir), but since they use `$(dirname "$0")/..`, they already resolve correctly.

5. **Create the root orchestrator Makefile**.

6. **Verify both components independently**, then from root.

---

### Scenario 5: Rename a Component Directory

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
	backend/make/build.sh

# New:
build:
	api-server/make/build.sh
```

3. **Update CI/CD configs** — `.github/workflows/`, `Dockerfile`, etc.

4. **Verify**:
```bash
make build && make test
```

---

## Reorganization Checklist

### Before
- [ ] `git status` is clean
- [ ] All tests pass
- [ ] Component names decided (domain-descriptive, not `backend/`, `src/`)

### During
- [ ] Move files to `<component>/`
- [ ] Never move `.venv/` — recreate it inside the component
- [ ] Fix ROOT_DIR only if scripts moved to a different depth
- [ ] Root Makefile has no language-specific build logic — global targets (deploy, ci, release) are allowed

### After
- [ ] `go test ./...` passes inside each Go component
- [ ] `make test` passes inside each Python component
- [ ] `make test` passes from git root
- [ ] No `go.mod` at git root
- [ ] No `pyproject.toml` or `.venv` at git root
- [ ] No `bin/` at git root
- [ ] Root Makefile has no language-specific logic
- [ ] Commit with clear message

---

## Rules

- **Never move `.venv/`** — symlinks inside it are absolute; always recreate it in the new location
- **Never add language files to the git root** — not even temporarily
- **Rename directories with `git mv`**, not `mv`, to preserve history
- **Reorganize before adding new features** — separate commits for structure vs. behavior
- **Root Makefile must not contain language-specific build or test logic** — if you catch yourself writing `go build` or `pip install` directly in a root target, move it to the component Makefile. Global targets like `deploy`, `ci`, `release`, and `infra-apply` are fine at root.

---

## Chaining

After completing all steps above:

1. **For each Go component** — invoke the `go-skeleton` skill to verify and fix the internal structure
2. **For each Python component** — invoke the `python-skeleton` skill to verify and fix the internal structure
3. **Check the root Makefile** — if it does not follow the `makefile-skeleton` standard, invoke the `makefile-skeleton` skill
4. **Check the root READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill
5. **Commit the changes** — invoke the `git-commit-suggest` skill to stage and commit the reorganization
