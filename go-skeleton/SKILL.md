---
name: go-skeleton
description: Standard Go project structure following community conventions (cmd/internal/pkg) inside a source subfolder. Creates from scratch or reorganizes existing projects.
mode: agent
category: go
shared: true
---

# Go Skeleton

Unified skill for organizing Go projects following modern Go community conventions. Uses the **structured layout** (cmd/internal/pkg) inside a source subfolder named after the project.

## Layout

> **Source code lives in `projectname/` subfolder with cmd/internal/pkg structure. Project root contains only infrastructure.**

```
project-name/              ← root: infrastructure only
├── go.mod                 ← module definition
├── go.sum
├── Makefile               ← orchestration
├── .make/                  ← build/test/install scripts
├── bin/                   ← compiled binaries (gitignored)
├── cfg/                   ← runtime config (optional)
├── docs/                  ← documentation (optional)
├── README.md
└── projectname/           ← ALL source code here (cmd/internal/pkg)
    ├── cmd/
    │   └── appname/
    │       └── main.go    ← entry point (package main, ≤50 lines)
    ├── internal/           ← private packages
    │   ├── config/
    │   │   └── config.go
    │   └── processor/
    │       └── processor.go
    └── pkg/                ← public packages (omit if none)
        └── types/
            └── types.go
```

**Key principle**: Source code is separated from project infrastructure. No `.go` files at the project root — all source belongs in `projectname/`.

## Package Organization

- **`cmd/appname/`** — Entry points (`package main`)
- **`internal/`** — Private packages (not importable outside module)
- **`pkg/`** — Public packages (importable externally, omit if none)

Import hierarchy: `cmd/ → internal/ → pkg/`. Never `pkg/ → internal/`.

## Context Detection

Before starting, determine the context:

1. **Check for `go.mod`** in the current directory (or the target directory):
   ```bash
   find . -name "go.mod" -not -path "./vendor/*" -maxdepth 3
   ```

2. **Establish the project root**:
   - If `go.mod` is at `.` (CWD) → project root is CWD.
   - If `go.mod` is in a subdirectory (e.g., `./myproject/go.mod`) → **the project root is that subdirectory**. All subsequent commands (file moves, package updates, Makefile edits) must be executed relative to that subdirectory. Treat it as if you had `cd myproject/` before starting.
   - If multiple `go.mod` files are found → ask the user which project to reorganize.

3. **If `go.mod` exists** → this is an existing project that needs reorganization. Follow **## Migrating an Existing Go Project** below.

4. **If `go.mod` does not exist** (or the directory is empty/new) → this is a new project being created from scratch. Follow **## Creating from Scratch** below.

5. **Always follow ## Canonical Layout** — it is the target structure for both flows.

6. **Follow ## Patterns** when the task involves testing patterns, error handling, anti-patterns, or code organization decisions.

---

## Migrating an Existing Go Project

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the project looks "mostly correct".**

### Mandatory Checklist

#### Before
- [ ] `git status` is clean
- [ ] All tests pass (`go test ./...`)
- [ ] Understand current structure

#### During
- [ ] Move files to correct locations
- [ ] Update package declarations
- [ ] Update imports (both moved and consuming code)
- [ ] Run `gofmt` on changed files
- [ ] Tests still pass after each move

#### After
- [ ] `go test ./...` passes
- [ ] `go build -o bin/project-name ./projectname/cmd/appname/` succeeds
- [ ] `go mod tidy` is clean
- [ ] No unused imports
- [ ] All `.go` files inside `projectname/`
- [ ] No `.go` files at project root
- [ ] Single Makefile at root
- [ ] `.make/` scripts present and executable
- [ ] `projectname/cmd/appname/main.go` ≤ 50 lines
- [ ] `bin/` in `.gitignore`
- [ ] Commit with clear message

### Rules

- **Never mix reorganization with logic changes** — reorganize first, then modify behavior in a separate commit
- **Move one thing at a time** — move a file, update package declarations, verify tests, repeat
- **All `.go` files belong in `projectname/`** — no Go source at project root
- **Only move existing files** — do not create new `.go` files that did not exist before
- **Keep `projectname/cmd/appname/main.go` thin** — ≤50 lines, flag parsing + delegation only
- **No generic file names** — `utils.go`, `helpers.go`, `common.go` must be renamed to domain-specific names

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

### Migration Scenarios

#### Scenario 1: Migrate from src/ Layout

**Symptom**: `go.mod` is inside `src/`, dual Makefiles, `.make/` scripts use `cd "$ROOT_DIR/src"`.

**Before**:
```
project/
├── Makefile              ← Delegates with make -C src
├── bin/
├── .make/
│   └── build.sh          ← cd "$ROOT_DIR/src" && go build ...
└── src/
    ├── Makefile           ← Second Makefile
    ├── go.mod
    ├── go.sum
    ├── cmd/
    └── internal/
```

**Steps**:

1. **Create the named directory and move Go module files**:
```bash
mkdir -p projectname bin
mv src/go.mod .
mv src/go.sum .
```

2. **Move all source files into projectname/**:
```bash
mv src/cmd/project-name/*.go projectname/cmd/projectname/
mv src/internal/processor/*.go projectname/internal/processor/
mv src/internal/config/*.go projectname/internal/config/
```

3. **Update package declarations** — they should already be correct (not `package main`):
```bash
# Verify packages are correct (should be processor, config, etc.)
grep "^package " projectname/internal/processor/processor.go
grep "^package " projectname/internal/config/config.go
```

4. **Update imports** — should already be using module paths:
```go
// Already correct in src/cmd/project-name/main.go:
import "github.com/user/project/internal/processor"

// Should remain the same after move
```

5. **Remove src/ and its Makefile**:
```bash
rm src/Makefile
rm -rf src/
```

6. **Replace root Makefile** — remove `make -C src` delegation:
```makefile
# Before:
lint:
	make -C src lint

# After:
lint:
	go vet ./...
```

7. **Update .make/build.sh**:
```bash
# Before: go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
# After:
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./projectname/cmd/$BINARY_NAME"
```

8. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./projectname/cmd/project-name/
```

---

#### Scenario 2: Flat Go Files at Root

**Symptom**: All `.go` files in project root, mixed with `Makefile`, `README.md`, `go.mod`.

**Before**:
```
project-name/
├── go.mod
├── main.go        ← mixed with project files
├── processor.go
└── types.go
```

**Steps**:

1. **Create the named package directory**:
```bash
mkdir -p projectname/bin projectname/cmd/appname
```

2. **Move main.go to cmd/appname/**:
```bash
mv main.go projectname/cmd/appname/main.go
```

3. **Create internal/pkg directories and move other files**:
```bash
mkdir -p projectname/internal/processor projectname/pkg/types
mv processor.go projectname/internal/processor/processor.go
mv types.go projectname/pkg/types/types.go
```

4. **Update package declarations**:
```bash
# projectname/cmd/appname/main.go should be package main
# projectname/internal/processor/processor.go should be package processor
# projectname/pkg/types/types.go should be package types
```

5. **Update imports in main.go**:
```go
// Old (if there were imports):
import "./processor"  // Wrong

// New:
import "github.com/user/project/internal/processor"
import "github.com/user/project/pkg/types"
```

6. **Update Makefile and .make/build.sh** (see ## Canonical Layout below)

7. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./projectname/cmd/appname/
```

---

#### Scenario 3: Monolithic main.go

**Symptom**: `projectname/main.go` with 200+ lines of business logic.

**Steps**:

1. **Create internal/pkg directories**:
```bash
mkdir -p projectname/internal/processor projectname/internal/config projectname/pkg/types
```

2. **Create focused files** inside `projectname/internal/`:
```bash
touch projectname/internal/processor/processor.go
touch projectname/internal/processor/processor_test.go
touch projectname/internal/config/config.go
touch projectname/pkg/types/types.go
```

3. **Move types** to `projectname/pkg/types/types.go` (package types)

4. **Move business logic** to `projectname/internal/processor/processor.go` (package processor)

5. **Move config logic** to `projectname/internal/config/config.go` (package config)

6. **Slim down `projectname/cmd/appname/main.go`** to ≤50 lines: flag parsing → call functions → handle errors

7. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./projectname/cmd/appname/
```

---

#### Scenario 4: Missing .make/ Scripts and Makefile

**Symptom**: Project has Go code but no automation.

**Steps**:

1. **Create .make/build.sh**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
CMD_PATH="./projectname/cmd/appname"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "$CMD_PATH"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

2. **Create .make/test.sh**:
```bash
#!/bin/bash
set -euo pipefail
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"
go test -v ./...
```

3. **Create .make/install.sh and .make/uninstall.sh** (see ## Canonical Layout below)

4. **Make executable**:
```bash
chmod +x .make/*.sh
```

5. **Create/replace Makefile** at project root (see ## Canonical Layout below for the full template):
```makefile
MAKEFLAGS += --no-print-directory
.PHONY: build test lint fmt clean install uninstall

build:
	./.make/build.sh

test:
	./.make/test.sh

lint:
	go vet ./...

fmt:
	go fmt ./...

clean:
	go clean
	rm -rf bin/

install: build
	./.make/install.sh

uninstall:
	./.make/uninstall.sh
```

---

#### Scenario 5: Duplicated Code Across Files

**Symptom**: Same logic copy-pasted across multiple files inside `projectname/internal/` or `projectname/pkg/`.

**Steps**:

1. **Compare duplicated code**:
```bash
grep -A 10 "func validatePath" projectname/internal/fetch.go
grep -A 10 "func validatePath" projectname/internal/export.go
```

2. **Extract to a shared file** inside appropriate internal/pkg directory:
```bash
touch projectname/internal/validation/validation.go
```

3. **Move the common implementation** to `projectname/internal/validation/validation.go`

4. **Update all callsites** to use the shared function with proper imports

5. **Delete duplicate code** from original files

6. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./projectname/cmd/appname/
```

---

#### Scenario 6: Generic File Names

**Symptom**: Files named `utils.go`, `helpers.go`, `common.go` with mixed concerns.

**Steps**:

1. **Analyze what's in the file**:
```bash
grep "^func" projectname/internal/utils.go
```

2. **Group functions by domain** — string ops, time ops, path ops, etc.

3. **Create domain-specific files** inside appropriate internal/pkg directories:
```bash
touch projectname/internal/stringutil/stringutil.go
touch projectname/internal/timeutil/timeutil.go
```

4. **Move functions** to appropriate files with correct package names

5. **Delete the old generic file**:
```bash
rm projectname/internal/utils.go
```

6. **Update imports in all consuming files**

7. **Verify**:
```bash
go test ./...
go build -o bin/project-name ./projectname/cmd/appname/
```

---

## Creating from Scratch

### Steps

1. **Determine `projectname`** — strip hyphens from folder name: `my-project` → `myproject`

2. **Create the module**:
```bash
go mod init github.com/user/project-name
```

3. **Create directories and entry point**:
```bash
mkdir -p projectname/bin make projectname/cmd/appname
cat > projectname/cmd/appname/main.go <<'EOF'
package main

import (
	"flag"
	"fmt"
	"log"
	"os"

	"github.com/user/project/internal/config"
)

func main() {
	flag.Usage = func() {
		fmt.Fprintf(os.Stderr, "Usage: project-name [flags]\n\n")
		flag.PrintDefaults()
	}
	flag.Parse()

	if err := run(); err != nil {
		log.Fatal(err)
	}
}

func run() error {
	fmt.Println("hello")
	return nil
}
EOF
```

4. **Create `.make/build.sh`**:
```bash
cat > .make/build.sh <<'EOF'
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
CMD_PATH="./projectname/cmd/appname"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "$CMD_PATH"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
EOF
chmod +x .make/build.sh
```

5. **Create `.make/test.sh`**:
```bash
cat > .make/test.sh <<'EOF'
#!/bin/bash
set -euo pipefail
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"
go test -v ./...
EOF
chmod +x .make/test.sh
```

6. **Create root Makefile** (see ## Canonical Layout for full template)

7. **Verify**:
```bash
go build -o bin/project-name ./projectname/cmd/appname/
./bin/project-name
```

---

### File Organization Patterns

#### Naming Files by Function

Name files by what they do, not generic names:

```go
// internal/detection/detection.go
func DetectOS() {...}
func DetectArchitecture() {...}

// internal/validation/validation.go
func ValidateInput() {...}

// pkg/types/types.go
type Config struct {...}
type Result struct {...}
```

Avoid: `utils.go`, `helpers.go`, `common.go`

#### Test Files

Tests live in the same directory as source:

```
projectname/
├── internal/processor/
│   ├── processor.go        ← Implementation
│   └── processor_test.go   ← Tests for processor.go
├── cmd/appname/
│   ├── main.go             ← Entry point
│   └── main_test.go        ← Integration tests
├── pkg/types/
│   ├── types.go            ← Type definitions
│   └── types_test.go       ← Tests for types
```

#### Types Organization

**Centralized**:
```
projectname/
├── pkg/types/
│   └── types.go            ← All shared types here
├── internal/config/
│   └── config.go           ← Config loading logic
│   └── config_test.go
```

**Distributed (for larger files)**:
```
projectname/
├── internal/processor/
│   ├── processor.go        ← Type Processor + logic
│   └── processor_test.go
├── internal/validator/
│   ├── validator.go        ← Type Validator + logic
│   └── validator_test.go
```

### Key Principles

#### 1. Clear Separation of Concerns

- **`projectname/`** = All source code (cmd/internal/pkg)
- **`cfg/`** = Runtime configuration defaults (optional)
- **`.make/`** = Development automation scripts
- **`bin/`** = Compiled outputs (don't commit)
- **`testdata/`** = Test fixtures and data
- Project root = infrastructure only: `go.mod`, `Makefile`, `README.md`

#### 2. Package Names

- `cmd/appname/` uses `package main`
- `internal/` directories use descriptive package names (e.g., `package config`, `package processor`)
- `pkg/` directories use descriptive package names (e.g., `package types`)

#### 3. Testing

```
projectname/pkg/types/types.go          → Implementation
projectname/pkg/types/types_test.go     → Tests (same package)
testdata/                                → Test fixtures
```

Run with `make test` or `go test ./...`

#### 4. Documentation

- **README.md** — English documentation
- **README-PT.md** — Portuguese documentation

#### 5. Go Module Location

- **`go.mod`** — Always at project root
- **`go.sum`** — Always at project root
- Single root Makefile for all operations

### Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add source file | `projectname/internal/domain/file.go` | Package = domain |
| Add tests | `projectname/internal/domain/file_test.go` | Same package, same directory |
| Add default configuration | `cfg/config.yaml` | Only if config needed |
| Add build script | `.make/script-name.sh` | Delegates to Go commands |
| Add test fixtures | `testdata/` | JSON, YAML, CSV, etc |
| Add public type | `projectname/pkg/types/types.go` | Exported, importable externally |

### When to Create a New File

**Create a new file inside `projectname/internal/` or `projectname/pkg/` if:**
- Logic is complex (100+ lines)
- Logic has a clear, distinct domain (parsing, validation, formatting)
- Tests for that logic would be clearer in a separate `_test.go` file

**Keep in cmd/appname/main.go if:**
- It's the entry point and flag parsing
- It's simple glue code (≤50 lines)

### Example: Adding a New Feature

**Scenario**: Add file validation capability to existing project

```
Step 1: Create internal/package
  mkdir -p projectname/internal/validation
  touch projectname/internal/validation/validation.go
  touch projectname/internal/validation/validation_test.go

Step 2: Add types if needed
  mkdir -p projectname/pkg/types
  touch projectname/pkg/types/types.go

Step 3: Add errors if needed
  touch projectname/internal/errors/errors.go

Step 4: Call from cmd/appname/main.go
  // In projectname/cmd/appname/main.go:
  import "github.com/user/project/internal/validation"
  
  if err := validation.ValidateFile(inputFile); err != nil {
      log.Fatal(err)
  }

Step 5: Add tests
  go test ./...

Step 6: Verify structure
  tree -I 'vendor|go.sum' -L 3
```

### Example Project Structures

#### Simple CLI Tool

```
makalu/
├── Makefile
├── go.mod
├── bin/
├── cfg/                  (optional)
├── .make/
│   ├── build.sh
│   └── test.sh
├── README.md
└── makalu/               ← all source (cmd/internal/pkg)
    ├── cmd/makalu/
    │   ├── main.go
    │   └── main_test.go
    ├── internal/discover/
    │   ├── discover.go
    │   └── discover_test.go
    ├── internal/catalog/
    │   ├── catalog.go
    │   └── catalog_test.go
    ├── internal/suggest/
    │   ├── suggest.go
    │   └── suggest_test.go
    ├── internal/errors/
    │   └── errors.go
    └── pkg/types/
        └── types.go
```

#### Larger Application

```
app/
├── Makefile
├── go.mod
├── go.sum
├── bin/
├── cfg/
│   └── config.yaml
├── .make/
│   ├── build.sh
│   ├── test.sh
│   └── deploy.sh
├── LICENSE
├── README.md
├── README-PT.md
├── testdata/
│   ├── input/
│   │   └── valid.json
│   └── expected/
│       └── output.json
└── app/                  ← all source (cmd/internal/pkg)
    ├── cmd/app/
    │   ├── main.go
    │   └── main_test.go
    ├── internal/server/
    │   ├── server.go
    │   └── server_test.go
    ├── internal/handlers/
    │   ├── handlers.go
    │   └── handlers_test.go
    ├── internal/config/
    │   ├── config.go
    │   └── config_test.go
    ├── internal/storage/
    │   ├── storage.go
    │   └── storage_test.go
    ├── internal/errors/
    │   └── errors.go
    └── pkg/types/
        └── types.go
```

### Best Practices

**Organizing projectname/ files**:
- `cmd/appname/main.go` ≤50 lines — flag parsing + delegation only
- One package per domain concern: `internal/processor/`, `internal/config/`, `internal/storage/`
- `internal/errors/` for all custom error types
- `pkg/types/` for shared type definitions

**File size guidelines**:
- `cmd/appname/main.go` ≤ 50 lines
- Other files ≤ 200 lines (split if larger)
- Function ≤ 50 lines (guideline, not rule)

**Code quality**:
- Run `gofmt` before commits
- Use linters (golangci-lint)
- Write table-driven tests
- Document exported functions (godoc)
- Keep error chains intact with `%w`

### Development Workflows

```bash
# Build
make build
# or directly
go build -o bin/project-name ./projectname/cmd/appname/
```

```bash
# Test
make test
# or directly
go test -v ./...
go test -coverprofile=coverage.out ./...
```

```bash
# Install / uninstall
make install
make uninstall
```

```bash
# Standard daily loop
make fmt                     # Format code
make lint                    # Run linter
make test                    # Run tests
make build                   # Build binary
./bin/project-name --help    # Test it
```

---

## Canonical Layout

Canonical target structure for all Go projects. `go.mod` lives at the project root. All source code lives in a directory named after the project (`projectname/`) with cmd/internal/pkg structure. A single root Makefile handles orchestration, while `.make/` scripts do the actual build/test work.

**Key principle**: Source code is separated from project infrastructure. No `.go` files at the project root — all source belongs in the named package directory.

### Standard Project Layout

```
project-name/
├── Makefile                      ← Single Makefile (orchestrates everything)
│
├── go.mod                        ← Go module definition (ALWAYS at root)
├── go.sum                        ← Dependency checksums
│
├── bin/                          ← Compiled binaries (never commit)
│   └── project-name              ← Executable after build
│
├── cfg/                          ← Configuration files (optional)
│   └── config.yaml               ← Default configuration
│
├── .make/                          ← Automation scripts
│   ├── build.sh                  ← Build script (go build)
│   ├── test.sh                   ← Test script
│   ├── install.sh                ← Install script
│   └── uninstall.sh              ← Uninstall script
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
└── projectname/                  ← ALL source code (named after project, no hyphens)
    ├── cmd/
    │   └── appname/
    │       ├── main.go           ← Entry point (package main)
    │       └── main_test.go      ← Integration tests
    ├── internal/                  ← Private packages
    │   ├── config/
    │   │   ├── config.go
    │   │   └── config_test.go
    │   ├── processor/
    │   │   ├── processor.go
    │   │   └── processor_test.go
    │   └── errors/
    │       └── errors.go
    └── pkg/                       ← Public packages (omit if none)
        └── types/
            ├── types.go
            └── types_test.go
```

> The directory name matches the project folder name with hyphens removed: `project-name` → `projectname`. Build: `go build -o bin/project-name ./projectname/cmd/appname/`.

### Core Directories

#### `bin/` — Compiled Binaries

**Purpose**: Store executable binaries after compilation.

```bash
bin/project-name              ← Compiled binary
./bin/project-name --help
```

**Note**: Add `bin/` to root `.gitignore` (build artifacts).

#### `cfg/` — Configuration Files (Optional)

**When to use**:
- Project requires runtime configuration
- Need environment-specific configs (dev, staging, prod)
- Simple CLI with only flags → skip this

```yaml
# cfg/config.yaml
log_level: info
port: 8080
database_url: postgres://localhost/db
timeout: 30
```

#### `.make/` — Automation Scripts

Scripts execute Go commands directly — do not delegate to make. Always start with `set -euo pipefail`.

```bash
.make/
├── build.sh       # Build binary to bin/
├── test.sh        # Run all tests
├── install.sh     # Install to ~/.local/bin
└── uninstall.sh   # Remove from ~/.local/bin
```

**`.make/build.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
CMD_PATH="./projectname/cmd/appname"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "$CMD_PATH"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

**`.make/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cd "$ROOT_DIR"
go test -v ./...
```

**`.make/install.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/bin/$BINARY_NAME" "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

**`.make/uninstall.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"

rm -f "$INSTALL_DIR/$BINARY_NAME"
echo "Removed: $INSTALL_DIR/$BINARY_NAME"
```

#### `projectname/` — All Source Code

All `.go` files live here with cmd/internal/pkg structure.

```
projectname/
├── cmd/
│   └── appname/
│       ├── main.go           ← Entry point (≤50 lines)
│       └── main_test.go      ← Integration tests
├── internal/                  ← Private packages
│   ├── config/
│   │   ├── config.go         ← Configuration loading
│   │   └── config_test.go
│   ├── processor/
│   │   ├── processor.go      ← Core logic
│   │   └── processor_test.go ← Unit tests
│   └── errors/
│       └── errors.go         ← Custom error types
└── pkg/                       ← Public packages
    └── types/
        ├── types.go          ← Shared type definitions
        └── types_test.go
```

| Directory | Purpose |
|-----------|---------|
| `cmd/appname/` | Entry point, parse flags, call functions. Keep ≤50 lines |
| `internal/` | Private packages, not importable outside module |
| `pkg/` | Public packages, importable externally (omit if none) |

**Key principle**: All code lives here. Split by package (domain), not by flat structure.

#### `testdata/` — Test Fixtures

```
testdata/
├── input/
│   ├── valid.json
│   └── invalid.json
│
└── expected/
    ├── output.json
    └── error.txt
```

### Single Makefile

One Makefile at the project root. Delegates build/test to `.make/` scripts.

```makefile
MAKEFLAGS += --no-print-directory

.PHONY: build test lint fmt clean install uninstall help

BINARY_NAME := project-name

build:
	./.make/build.sh

test:
	./.make/test.sh

lint:
	go vet ./...

fmt:
	go fmt ./...

clean:
	go clean
	rm -rf bin/

install: build
	./.make/install.sh

uninstall:
	./.make/uninstall.sh

help:
	@echo "Usage: make <target>"
	@echo ""
	@echo "Targets:"
	@echo "  build      Build the binary"
	@echo "  test       Run tests"
	@echo "  lint       Run linter"
	@echo "  fmt        Format code"
	@echo "  clean      Remove build artifacts"
	@echo "  install    Install to ~/.local/bin"
	@echo "  uninstall  Remove from ~/.local/bin"
```

**Key principles**:
- Root Makefile is the single entry point for all operations
- `build` and `test` delegate to `.make/` scripts
- `lint`, `fmt`, `clean` run Go commands directly

### Go Module and Imports

**`go.mod`** — always at project root.

**Standard (required)**:
```
module github.com/carlosrabelo/project-name
```

**Exception: Simple path (for throwaway/internal-only projects)**:
```
module project-name
```

**Key point**: `go.mod` at root, source in `projectname/`, no `.go` files at root.

### .gitignore

```
# Build artifacts
bin/
project-name          ← replace with the actual binary name; catches go build without -o

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Go
coverage.out
coverage.html
```

---

## Package Placement Rules

| Code | Location | Rule |
|------|----------|------|
| CLI flags + wiring | `cmd/appname/main.go` | ≤50 lines, zero business logic |
| Domain logic | `internal/domain/` | Private, not importable outside module |
| Shared types / interfaces | `pkg/types/` | Only if external consumers exist |
| Test fixtures | `testdata/` | Never inside `internal/` |

**Forbidden**: `pkg/` importing `internal/` (leaks private details).

---

## Patterns

### Testing Patterns

#### Table-Driven Tests

Go's idiomatic approach to testing multiple cases:

```go
func TestValidateInput(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    bool
        wantErr bool
    }{
        {
            name:    "valid input",
            input:   "example.txt",
            want:    true,
            wantErr: false,
        },
        {
            name:    "empty input",
            input:   "",
            want:    false,
            wantErr: true,
        },
        {
            name:    "invalid extension",
            input:   "example.doc",
            want:    false,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ValidateInput(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("unexpected error: %v", err)
            }
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

#### Test Fixtures

Use `testdata/` for external test files:

```go
func TestProcessFile(t *testing.T) {
    data, err := ioutil.ReadFile("testdata/input/valid.json")
    if err != nil {
        t.Fatalf("failed to load test fixture: %v", err)
    }

    result, err := ProcessData(data)
    // assertions...
}
```

#### Coverage Targets

```bash
# Run with coverage
go test -coverprofile=coverage.out ./...

# View HTML report
go tool cover -html=coverage.out -o coverage.html
```

- **80%+** overall coverage
- **100%** for critical paths
- Don't obsess over coverage — test behavior, not lines

### Error Handling Patterns

#### Custom Error Types

Define custom error types in appropriate internal package:

```go
// internal/errors/errors.go
package errors

import "fmt"

// ValidationError indicates invalid input
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed on %s: %s", e.Field, e.Message)
}

// ProcessingError indicates processing failed
type ProcessingError struct {
    Reason string
    Cause  error
}

func (e *ProcessingError) Error() string {
    return fmt.Sprintf("processing failed: %s", e.Reason)
}
```

#### Error Wrapping

Use `fmt.Errorf("%w", err)` to wrap errors:

```go
// Good: preserves error chain
result, err := loadConfig("config.yaml")
if err != nil {
    return fmt.Errorf("failed to load configuration: %w", err)
}

// Bad: loses error context
if err != nil {
    return err  // Caller loses context
}

// Bad: doesn't preserve error type
if err != nil {
    return fmt.Errorf("error: %v", err)  // Can't use errors.Is()
}
```

#### Error Checking

```go
// Check specific error types
if errors.Is(err, os.ErrNotExist) {
    // handle missing file
}

// Check custom types
var ve *ValidationError
if errors.As(err, &ve) {
    log.Printf("Validation failed on field: %s", ve.Field)
}
```

#### When to Log vs Return

**Return error if**: caller should decide, error can be recovered, error affects program logic.

**Log error if**: unrecoverable situation, informational logging, error is already being returned (don't double-log).

```go
// In internal/processor/processor.go
func process(input string) (Result, error) {
    if input == "" {
        return nil, &errors.ValidationError{Field: "input", Message: "empty"}
    }

    data, err := loadData(input)
    if err != nil {
        return nil, fmt.Errorf("load failed: %w", err)
    }

    return data, nil
}

// In cmd/appname/main.go
func main() {
    result, err := processor.Process("file.txt")
    if err != nil {
        log.Fatalf("Processing failed: %v", err)
    }

    fmt.Println(result)
}
```

### Anti-Patterns: What NOT to Do

#### Anti-Pattern 1: Go Source Files at Project Root

Leaving `.go` files at the project root mixes source code with `go.mod`, `Makefile`, `README.md` — no clear boundary between code and infrastructure.

```
# BAD: .go files at root alongside project files
project-name/
├── main.go           ← code mixed with...
├── processor.go      ← ...go.mod, Makefile, README
├── go.mod
└── Makefile

# GOOD: all .go files inside named directory
project-name/
├── projectname/      ← cmd/internal/pkg structure
│   ├── cmd/appname/
│   ├── internal/
│   └── pkg/
├── go.mod
└── Makefile
```

#### Anti-Pattern 2: go.mod in a Subdirectory

```
# BAD: go.mod in src/ (legacy GOPATH-era pattern)
project/
├── Makefile
└── src/
    ├── go.mod          ← Wrong location
    └── projectname/

# GOOD: go.mod at project root
project/
├── Makefile
├── go.mod              ← Correct: project root
└── projectname/
    ├── cmd/appname/
    ├── internal/
    └── pkg/
```

#### Anti-Pattern 3: Logic in main.go

```go
// BAD: 200 lines of business logic in main.go
package main

func main() {
    data, _ := ioutil.ReadFile("config.json")
    // ... 200 more lines of logic
}
```

```go
// GOOD: main.go delegates to other packages
package main

import (
    "log"
    
    "github.com/user/project/internal/config"
    "github.com/user/project/internal/processor"
)

func main() {
    cfg, err := config.Load("config.json")
    if err != nil {
        log.Fatal(err)
    }

    items, err := processor.Process(cfg)
    if err != nil {
        log.Fatal(err)
    }
}
```

#### Anti-Pattern 4: Ignoring Errors Silently

```go
// BAD: Silent failures
data, _ := ioutil.ReadFile("config.json")  // Error ignored!
json.Unmarshal(data, &config)              // May fail

// GOOD: Handle or propagate errors
data, err := ioutil.ReadFile("config.json")
if err != nil {
    return fmt.Errorf("load config: %w", err)
}

if err := json.Unmarshal(data, &config); err != nil {
    return fmt.Errorf("parse config: %w", err)
}
```

#### Anti-Pattern 5: Generic File Names

```go
// BAD: Generic names inside internal/
projectname/
└── internal/
    ├── utils.go
    ├── helpers.go
    └── common.go

// GOOD: Specific, domain-driven names
projectname/
└── internal/
    ├── config.go      // Configuration loading
    ├── processor.go   // Data processing
    ├── discovery.go   // System discovery
    └── validation.go  // Input validation
```

#### Anti-Pattern 6: Panicking in Production Code

```go
// BAD: Panics crash the program
func LoadConfig(file string) Config {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        panic("Config not found!")
    }
}

// GOOD: Return errors
func LoadConfig(file string) (Config, error) {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        return Config{}, fmt.Errorf("config not found: %w", err)
    }
    return config, nil
}
```

`panic()` is acceptable only in `cmd/appname/main.go` for truly unrecoverable scenarios.

#### Anti-Pattern 7: Mixing Concerns in One Package

```
# BAD: Multiple concerns in one package
projectname/
└── internal/
    └── everything.go  // Config, validation, processing, output formatting

# GOOD: Separate by concern in different packages
projectname/
└── internal/
    ├── types/       // Type definitions
    ├── config/      // Configuration loading
    ├── validation/  // Validation logic
    ├── processor/   // Core processing
    └── output/      // Result formatting
```

#### Anti-Pattern 8: Not Testing Edge Cases

```go
// BAD: Only happy path
func TestProcess(t *testing.T) {
    result := Process("valid input")
    if result != "expected" {
        t.Fail()
    }
}

// GOOD: Table-driven tests with edge cases
func TestProcess(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr bool
    }{
        {"valid", "good", "expected", false},
        {"empty", "", "", true},
        {"null", "null", "", true},
        {"unicode", "café", "expected", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Process(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("got = %v, want %v", got, tt.want)
            }
        })
    }
}
```

#### Anti-Pattern 9: Two Makefiles

```
# BAD: Dual Makefile hierarchy
project/
├── Makefile          ← Root: delegates with make -C src
└── src/
    └── Makefile      ← Second Makefile for Go targets

# GOOD: Single Makefile at root
project/
├── Makefile          ← Single Makefile handles everything
├── go.mod
└── projectname/
    ├── cmd/appname/
    ├── internal/
    └── pkg/
```

#### Anti-Pattern 10: Running `go build` Without `-o`

```bash
# BAD: binary lands in project root (or inside projectname/)
go build ./projectname/cmd/appname/

# GOOD: Always use make build or ./.make/build.sh
make build
# or
./.make/build.sh
```

The `.make/build.sh` script always uses `-o "$ROOT_DIR/bin/$BINARY_NAME"`, guaranteeing the binary goes to `bin/`.

As a safety net, add the binary name to `.gitignore`:
```gitignore
bin/
project-name          ← binary name without path, catches root-level builds
```

### Code Comments

**Language**: Always English, even if the rest of the project uses Portuguese.

**Godoc** (exported identifiers):
- First line starts with the identifier name: `// Process reads input from…`
- Complete sentence, ending with period
- Unexported functions follow the same format (lowercase name)

**Inline comments**:
- Explain **why**, not what — if the code is obvious, no comment needed
- Above the line (not end-of-line) for multi-word explanations

**All exported functions, types, and struct fields must have doc comments.**

---

## Monorepo Usage

This skill applies to whichever directory contains `go.mod` — that is the Go component root, regardless of where the git root is. The `projectname/` pattern with cmd/internal/pkg still applies inside each component.

```
monorepo/
└── my-tool/               ← component root (go.mod lives here)
    ├── mytool/            ← all source code (projectname/ pattern)
    │   ├── cmd/mytool/
    │   ├── internal/
    │   └── pkg/
    ├── bin/
    ├── go.mod
    └── Makefile
```

- `go.mod` lives at `<component>/`, not at the git root
- `<component>/.make/` scripts resolve `ROOT_DIR` to the component dir: `$(cd "$(dirname "$0")/.." && pwd)`
- The git root has a separate orchestrator Makefile — this is **not** Anti-Pattern 9 (Two Makefiles)
- When updating `.gitignore`, paths are relative to `<component>/`, not the git root

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

---

## Chaining

When creating a complete Go project from scratch, the full workflow involves these skills in order:

1. **`gitignore-skeleton`** — `.gitignore` with Go patterns (binaries, vendor, caches)
2. **`readme-skeleton`** — `README.md` with the standard structure (handles `README-PT.md` automatically)

After completing a migration:

1. **Check the Makefile** — verify it matches the standard in ## Canonical Layout above (opening lines, .PHONY, help pattern, .make/ delegation)
2. **Check the `.gitignore`** — if it is missing the AI Tools section or deviates from the standard, invoke the `gitignore-skeleton` skill
3. **Check the READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill
4. **Commit the changes** — invoke the `git-commit-suggest` skill to stage and commit the reorganization

## Related Skills

- **makefile-skeleton** — Generic Makefile structure conventions (opening lines, .PHONY, help pattern)
- **readme-skeleton** — Standard README content, section order, and bilingual conventions
- **gitignore-skeleton** — For bringing `.gitignore` up to the standard after reorganization
- **monorepo-skeleton** — For organizing multi-language monorepos with Go as one component
