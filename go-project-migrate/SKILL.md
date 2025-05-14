---
name: go-project-migrate
description: Reorganize existing Go projects to match the standard flat root structure (go.mod at root, cmd/, internal/, single Makefile, make/ scripts). Handles migration from src/ layout, monolithic code, and missing structure.
mode: manual
category: go
shared: true
---

# Go Project Migrate

Reorganize existing Go projects to match the standard structure defined in `go-project-create`. Invoke manually with `/go-project-migrate` when a project needs structural alignment.

## Overview

This skill transforms messy or legacy Go projects into the standard layout:

```
project/
├── Makefile          ← Single Makefile
├── go.mod            ← At project root (NEVER in src/)
├── go.sum
├── bin/              ← Compiled binaries (.gitignore)
├── make/              ← Automation scripts
│   ├── build.sh
│   ├── test.sh
│   ├── install.sh
│   └── uninstall.sh
├── cmd/              ← Executable entry points
│   └── project-name/
│       └── main.go
├── internal/         ← Internal packages
│   └── package/
│       ├── types.go
│       ├── logic.go
│       └── logic_test.go
└── testdata/         ← Test fixtures
```

---

## Before You Start

### Checklist

- ✅ All changes committed (`git status` clean)
- ✅ Tests passing (`go test ./...`)
- ✅ Understand what the project does
- ✅ Identify what exists and what's missing

### Diagnose the Project

```bash
# Where is go.mod?
find . -name "go.mod" -not -path "./vendor/*"

# What directories exist?
ls -d */ 2>/dev/null

# Where are the Go files?
find . -name "*.go" -not -path "./vendor/*" | head -20

# How big is main.go?
wc -l $(find . -name "main.go" -not -path "./vendor/*")
```

---

## Reorganization Scenarios

### Scenario 1: Migrate from src/ Layout

**Symptom**: `go.mod` is inside `src/`, dual Makefiles, `make/` scripts use `cd "$ROOT_DIR/src"`.

**Before**:
```
project/
├── Makefile              ← Delegates with make -C src
├── bin/
├── make/
│   └── build.sh          ← cd "$ROOT_DIR/src" && go build ...
└── src/
    ├── Makefile           ← Second Makefile
    ├── go.mod
    ├── go.sum
    ├── cmd/
    └── internal/
```

**Steps**:

1. **Move Go module files to root**:
```bash
mv src/go.mod .
mv src/go.sum .
```

2. **Move Go directories to root**:
```bash
mv src/cmd .
mv src/internal .
mv src/testdata .    # if exists
# mv src/*.go .      # if any exist at src/ root
```

3. **Remove src/ and its Makefile**:
```bash
rm src/Makefile
rmdir src
```

4. **Replace root Makefile** — remove `make -C src` delegation, use direct Go commands:
```makefile
# Before:
lint:
	make -C src lint

# After:
lint:
	go vet ./...
```

5. **Update make/ scripts** — change `cd "$ROOT_DIR/src"` to `cd "$ROOT_DIR"`:
```bash
# Before:
cd "$ROOT_DIR/src"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"

# After:
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
```

6. **Update .gitignore** — remove `src/` prefix:
```
# Before:
src/coverage.out

# After:
coverage.out
```

7. **Verify** — imports don't change because `src/` was never part of the module path:
```bash
go test ./...
go build ./cmd/project-name
```

---

### Scenario 2: Flat Go Files (No Structure)

**Symptom**: All `.go` files in project root, no `cmd/` or `internal/`.

**Before**:
```
project/
├── go.mod
├── main.go        ← 300+ lines
├── helpers.go
└── types.go
```

**Steps**:

1. **Create directory structure**:
```bash
mkdir -p cmd/project-name internal/core bin run
```

2. **Move entry point to cmd/**:
```bash
mv main.go cmd/project-name/main.go
```

3. **Extract business logic to internal/**:
   - Move types to `internal/core/types.go`
   - Move logic to `internal/core/logic.go`
   - Update package declarations from `package main` to `package core`
   - Export functions that `cmd/` needs (capitalize first letter)

4. **Update cmd/project-name/main.go** — keep it thin (≤50 lines):
```go
package main

import (
    "log"
    "github.com/user/project/internal/core"
)

func main() {
    // Flag parsing + call into internal/
}
```

5. **Add make/ scripts and Makefile** (see go-project-create)

6. **Delete old files from root**:
```bash
rm helpers.go types.go  # now in internal/
```

7. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./cmd/project-name
```

---

### Scenario 3: Monolithic main.go

**Symptom**: Single `cmd/project-name/main.go` with 200+ lines of business logic.

**Steps**:

1. **Identify concerns** in main.go:
   - Flag parsing / CLI setup
   - Configuration loading
   - Business logic
   - Output formatting
   - Error types

2. **Create internal package** for the domain:
```bash
mkdir -p internal/processor
touch internal/processor/{types,processor,processor_test,errors}.go
```

3. **Move types** to `internal/processor/types.go`

4. **Move business logic** to `internal/processor/processor.go`

5. **Add tests** in `internal/processor/processor_test.go`

6. **Slim down main.go** to ≤50 lines: flag parsing → call internal packages → handle errors

7. **Verify**:
```bash
go test ./...
go build ./cmd/project-name
```

---

### Scenario 4: Missing make/ Scripts and Makefile

**Symptom**: Project has Go code but no automation.

**Steps**:

1. **Create make/build.sh**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

2. **Create make/test.sh**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cd "$ROOT_DIR"
go test -v ./...
```

3. **Create make/install.sh and make/uninstall.sh** (see go-project-create)

4. **Make executable**:
```bash
chmod +x make/*.sh
```

5. **Create/replace Makefile** at project root:
```makefile
MAKEFLAGS += --no-print-directory
.PHONY: build test lint fmt clean install uninstall

build:
	./make/build.sh

test:
	./make/test.sh

lint:
	go vet ./...

fmt:
	go fmt ./...

clean:
	go clean
	rm -rf bin/

install: build
	./make/install.sh

uninstall:
	./make/uninstall.sh
```

---

### Scenario 5: Duplicated Code Across Commands

**Symptom**: Multiple `cmd/` directories with copied logic.

**Steps**:

1. **Compare duplicated files**:
```bash
diff cmd/command-a/shared.go cmd/command-b/shared.go
```

2. **Create shared internal package**:
```bash
mkdir -p internal/shared-concern
```

3. **Move common code** to `internal/shared-concern/`
   - Update package declaration
   - Export needed functions

4. **Update both commands** to import from `internal/`

5. **Delete duplicate files** from `cmd/`

6. **Verify**:
```bash
go test ./...
go build ./cmd/command-a
go build ./cmd/command-b
```

---

### Scenario 6: Generic Package Names

**Symptom**: Packages named `utils/`, `helpers/`, `common/`.

**Steps**:

1. **Analyze what's in the package**:
```bash
grep "^func" internal/utils/*.go
```

2. **Group functions by domain** — string ops, time ops, path ops, etc.

3. **Create domain-specific packages**:
```bash
mkdir -p internal/stringutil internal/timeutil
```

4. **Move functions** to appropriate packages, update package declarations

5. **Update all imports** across the codebase:
```bash
grep -r "internal/utils" --include="*.go"
```

6. **Delete old generic package**:
```bash
rm -rf internal/utils/
```

---

## Reorganization Checklist

### Before
- [ ] `git status` is clean
- [ ] All tests pass
- [ ] Understand current structure

### During
- [ ] Move files to correct locations
- [ ] Update package declarations
- [ ] Update imports (both moved and consuming code)
- [ ] Run `gofmt` on changed files
- [ ] Tests still pass after each move

### After
- [ ] `go test ./...` passes
- [ ] `go build ./cmd/...` succeeds
- [ ] `go mod tidy` is clean
- [ ] No unused imports
- [ ] `main.go` ≤ 50 lines
- [ ] Single Makefile at root
- [ ] `make/` scripts present and executable
- [ ] No `src/` directory
- [ ] `bin/` in `.gitignore`
- [ ] Commit with clear message

---

## Rules

- **Never mix reorganization with logic changes** — reorganize first, then modify behavior in a separate commit
- **Move one thing at a time** — move a file, update imports, verify tests, repeat
- **Import paths don't include src/** — if migrating from `src/`, imports stay the same
- **Keep main.go thin** — ≤50 lines, flag parsing + delegation only
- **No generic package names** — `utils/`, `helpers/`, `common/` must be renamed to domain-specific names

---

## Monorepo Usage

This skill applies to whichever directory contains `go.mod` — that is the Go project root, regardless of where the git root is.

When migrating a Go component inside a monorepo:

- Treat `<component>/` as the project root throughout all scenarios. `go.mod`, `Makefile`, `make/`, `cmd/`, and `internal/` all live there.
- `make/` scripts compute `ROOT_DIR` as `$(dirname "$0")/..`, resolving to `<component>/` — not the git root.
- The git root has a separate orchestrator Makefile — this is **not** a violation of the single-Makefile rule.
- When updating `.gitignore`, paths are relative to `<component>/`, not the git root.

See **monorepo-project-create** for the full monorepo layout and root Makefile patterns.

---

## Chaining

After completing all steps above:

1. **Check the Makefile** — if it exists but does not follow the `makefile-create` standard, invoke the `makefile-migrate` skill
2. **Check the READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-migrate` skill

---

## Related Skills

- **go-project-create** — The target structure this skill reorganizes toward
- **monorepo-project-create** — For organizing the git root when Go is one component among many
- **monorepo-project-migrate** — For migrating the repo itself into a monorepo layout
