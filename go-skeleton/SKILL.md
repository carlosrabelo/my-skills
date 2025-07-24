---
name: go-skeleton
description: Standard Go project structure (go.mod at root, cmd/, internal/, make/ scripts). Creates from scratch or reorganizes existing projects.
mode: agent
category: go
shared: true
---

# Go Skeleton

Unified skill for organizing Go projects following modern Go conventions. Handles both new projects and reorganization of existing ones.

## Context Detection

Before starting, determine the context:

1. **Check for `go.mod`** in the current directory (or the target directory):
   ```bash
   find . -name "go.mod" -not -path "./vendor/*" -maxdepth 3
   ```

2. **If `go.mod` exists** → this is an existing project that needs reorganization. Follow **## Migrating an Existing Go Project** below.

3. **If `go.mod` does not exist** (or the directory is empty/new) → this is a new project being created from scratch. Follow **## Creating from Scratch** below.

4. **Always follow ## Canonical Layout** — it is the target structure for both flows.

5. **Follow ## Patterns** when the task involves testing patterns, error handling, anti-patterns, or code organization decisions.

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
- [ ] `go build ./cmd/...` succeeds
- [ ] `go mod tidy` is clean
- [ ] No unused imports
- [ ] `main.go` ≤ 50 lines
- [ ] Single Makefile at root
- [ ] `make/` scripts present and executable
- [ ] No `src/` directory
- [ ] `bin/` in `.gitignore`
- [ ] Commit with clear message

### Rules

- **Never mix reorganization with logic changes** — reorganize first, then modify behavior in a separate commit
- **Move one thing at a time** — move a file, update imports, verify tests, repeat
- **Import paths don't include src/** — if migrating from `src/`, imports stay the same
- **Keep main.go thin** — ≤50 lines, flag parsing + delegation only
- **No generic package names** — `utils/`, `helpers/`, `common/` must be renamed to domain-specific names

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

#### Scenario 2: Flat Go Files (No Structure)

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
mkdir -p cmd/project-name internal/core bin
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

5. **Add make/ scripts and Makefile** (see ## Canonical Layout below)

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

#### Scenario 3: Monolithic main.go

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

#### Scenario 4: Missing make/ Scripts and Makefile

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

3. **Create make/install.sh and make/uninstall.sh** (see ## Canonical Layout below)

4. **Make executable**:
```bash
chmod +x make/*.sh
```

5. **Create/replace Makefile** at project root (see ## Canonical Layout below for the full template):
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

#### Scenario 5: Duplicated Code Across Commands

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

#### Scenario 6: Generic Package Names

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

## Creating from Scratch

### File Organization Patterns

#### Naming Files by Function

Name files by what they do, not generic names:

```go
// detection.go
func DetectOS() {...}
func DetectArchitecture() {...}

// validation.go
func ValidateInput() {...}

// types.go
type Config struct {...}
type Result struct {...}
```

Avoid: `utils.go`, `helpers.go`, `common.go`

#### Test Files

Tests live in the same package and directory as source:

```
internal/processor/
├── process.go          ← Implementation
├── process_test.go     ← Tests for process.go
├── types.go            ← Type definitions
└── errors.go           ← Error types

cmd/main/
├── main.go             ← Entry point
├── main_test.go        ← Integration tests
└── flags.go            ← Flag parsing
```

#### Types Organization

**Option 1: Centralized (for small packages)**
```
internal/config/
├── types.go            ← All types here
├── load.go             ← Config loading logic
└── load_test.go
```

**Option 2: Distributed (for larger packages)**
```
internal/processor/
├── types.go            ← Core types
├── process.go          ← Type Processor + logic
├── process_test.go
├── validator.go        ← Type Validator + logic
└── validator_test.go
```

### Key Principles

#### 1. Clear Separation of Concerns

- **`cmd/`** = Executable-specific logic and CLI interface
- **`internal/`** = Reusable, testable business logic
- **`cfg/`** = Runtime configuration defaults (optional)
- **`make/`** = Development automation scripts
- **`bin/`** = Compiled outputs (don't commit)
- **`testdata/`** = Test fixtures and data

#### 2. Package Naming

Package name matches directory name:

```
internal/setup/    → package setup
internal/config/   → package config
cmd/main/          → package main
```

#### 3. Testing

```
file.go              → Implementation
file_test.go         → Tests (same package)
testdata/            → Test fixtures
```

Run with `make test` or `go test ./...`

#### 4. Documentation

- **README.md** — English documentation
- **README-PT.md** — Portuguese documentation

#### 5. Visibility

- **`cmd/`** — Private to the executable
- **`internal/`** — Private to the project (Go enforces this)
- **`bin/`** — Build outputs (add to `.gitignore`)
- **`pkg/`** — Public (only if creating a library)

#### 6. Go Module Location

- **`go.mod`** — Always at project root
- **`go.sum`** — Always at project root
- Single root Makefile for all operations

### Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add new CLI command | `cmd/command-name/` | One binary per directory |
| Add reusable logic | `internal/package-name/` | Used by multiple commands |
| Add default configuration | `cfg/config.yaml` | Only if config needed |
| Add build script | `make/script-name.sh` | Delegates to Go commands |
| Add test fixtures | `testdata/` | JSON, YAML, CSV, etc |
| Add library for external use | `pkg/library-name/` | Only for publishable libs |

### When to Create a New Package

**Create a package if:**
- Code is used by multiple commands
- Logic is complex (100+ lines, distinct domain)
- Code benefits from being tested separately
- Clear, single responsibility

**Keep in cmd if:**
- Code is specific to one command
- Only used by one executable
- Simple glue code (flag parsing, coordination)

### Example: Adding a New Feature

**Scenario**: Add file validation capability to existing project

```
Step 1: Create internal package
  mkdir -p internal/validator

Step 2: Create files
  internal/validator/types.go           ← Type definitions
  internal/validator/validate.go        ← Validation logic
  internal/validator/validate_test.go   ← Tests
  internal/validator/errors.go          ← Error types

Step 3: Update command to use it
  cmd/main/main.go
  import "github.com/carlosrabelo/project/internal/validator"

  // In main():
  if err := validator.ValidateFile(inputFile); err != nil {
      log.Fatal(err)
  }

Step 4: Add tests
  go test ./...

Step 5: Verify structure
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
├── make/
│   ├── build.sh
│   └── test.sh
├── README.md
├── cmd/
│   └── makalu/
│       ├── main.go
│       ├── flags.go
│       └── main_test.go
└── internal/
    ├── discovery/
    │   ├── types.go
    │   ├── discover.go
    │   └── discover_test.go
    ├── inventory/
    │   ├── types.go
    │   ├── catalog.go
    │   └── catalog_test.go
    └── suggestion/
        ├── types.go
        ├── suggest.go
        └── suggest_test.go
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
├── make/
│   ├── build.sh
│   ├── test.sh
│   └── deploy.sh
├── LICENSE
├── README.md
├── README-PT.md
├── cmd/
│   ├── main-app/
│   │   ├── main.go
│   │   ├── flags.go
│   │   ├── commands.go
│   │   └── main_test.go
│   └── tools/
│       ├── main.go
│       └── main_test.go
├── internal/
│   ├── api/
│   │   ├── server.go
│   │   ├── handlers.go
│   │   ├── handlers_test.go
│   │   └── errors.go
│   ├── config/
│   │   ├── load.go
│   │   ├── load_test.go
│   │   └── types.go
│   └── storage/
│       ├── db.go
│       ├── db_test.go
│       ├── queries.go
│       └── types.go
└── testdata/
    ├── input/
    │   └── valid.json
    └── expected/
        └── output.json
```

### Best Practices

**Organizing cmd/ packages**:
- Keep `main.go` ≤50 lines
- Put flag parsing in `flags.go`
- Put core logic in domain-specific files
- Use `internal/` for reusable code
- Write integration tests in `main_test.go`

**Organizing internal/ packages**:
- One responsibility per package
- Group related types together in `types.go`
- Keep test coverage high (80%+)
- Document exported functions and types
- Use error types from `errors.go`

**File size guidelines**:
- `main.go` ≤ 50 lines
- Other files ≤ 200 lines (split if larger)
- Package = one clear concern
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
go build -o bin/project-name ./cmd/project-name
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

Canonical target structure for all Go projects. `go.mod` lives at the project root alongside `cmd/`, `internal/`, and `testdata/`. A single root Makefile handles orchestration, while `make/` scripts do the actual build/test work.

**Key principle**: This structure follows the official Go module layout (go.dev/doc/modules/layout). All Go projects — CLI tools, applications, and libraries — use the same pattern.

### Standard Project Layout

```
project-name/
├── Makefile                      ← Single Makefile (orchestrates everything)
│
├── go.mod                        ← Go module definition (ALWAYS at root)
├── go.sum                        ← Dependency checksums
│
├── bin/                          ← Compiled binaries
│   └── project-name              ← Executable after build
│
├── cfg/                          ← Configuration files (optional)
│   └── config.yaml               ← Default configuration
│
├── make/                          ← Automation scripts
│   ├── build.sh                  ← Build script (go build)
│   ├── test.sh                   ← Test script
│   ├── install.sh                ← Install script
│   └── uninstall.sh              ← Uninstall script
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
├── cmd/                          ← Executable entry points
│   └── project-name/             ← Main command
│       ├── main.go               ← Entry point
│       ├── main_test.go          ← Tests
│       └── *.go                  ← Implementation files
│
├── internal/                     ← Internal packages (not importable)
│   ├── package-one/              ← Reusable logic
│   │   ├── types.go              ← Type definitions
│   │   ├── logic.go              ← Core logic
│   │   ├── logic_test.go         ← Tests
│   │   └── errors.go             ← Error types
│   │
│   └── package-two/              ← Another package
│       ├── types.go
│       ├── handler.go
│       └── handler_test.go
│
└── testdata/                     ← Test fixtures and data
    ├── input.json
    └── expected.json
```

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

#### `make/` — Automation Scripts

Scripts execute Go commands directly — do not delegate to make. Always start with `set -euo pipefail`.

```bash
make/
├── build.sh       # Build binary to bin/
├── test.sh        # Run all tests
├── install.sh     # Install to ~/.local/bin
└── uninstall.sh   # Remove from ~/.local/bin
```

**`make/build.sh`**:
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

**`make/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cd "$ROOT_DIR"
go test -v ./...
```

**`make/install.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/bin/$BINARY_NAME" "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

**`make/uninstall.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"

rm -f "$INSTALL_DIR/$BINARY_NAME"
echo "Removed: $INSTALL_DIR/$BINARY_NAME"
```

#### `cmd/` — Command Executables

One subdirectory = one executable. Keep `main.go` ≤50 lines.

```
cmd/
├── main-command/         ← First executable
│   ├── main.go           ← Entry point (≤50 lines)
│   ├── flags.go          ← Flag parsing
│   ├── commands.go       ← Command routing
│   └── main_test.go      ← Integration tests
│
└── other-command/        ← Additional executable (if needed)
    ├── main.go
    ├── flags.go
    └── main_test.go
```

| File | Purpose |
|------|---------|
| `main.go` | Entry point, parse flags, call functions. Keep ≤50 lines |
| `flags.go` | Command-line flag definitions and parsing |
| `commands.go` | Command routing and coordination logic |
| `types.go` | Type definitions specific to this command |
| `*_test.go` | Tests (same package, one per source file) |

**Key principle**: Commands contain executable-specific logic. Reusable code goes in `internal/`.

#### `internal/` — Reusable Internal Packages

Code that's reusable within the project but not importable externally (Go enforces this).

```
internal/
├── discovery/           ← Example: Find/detect things
│   ├── types.go         ← Type definitions
│   ├── discover.go      ← Core logic
│   ├── discover_test.go ← Tests
│   └── errors.go        ← Error types
│
├── processor/           ← Example: Process data
│   ├── types.go
│   ├── process.go
│   └── process_test.go
│
└── config/              ← Example: Configuration
    ├── load.go
    ├── load_test.go
    └── types.go
```

**When to create a package**:
- Code is used by multiple commands
- Logic is complex (100+ lines, distinct domain)
- Code benefits from being tested separately
- Clear responsibility and single concern

**When to keep in cmd**:
- Code is specific to one command
- Only used by one executable
- Simple glue code (flag parsing, coordination)

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

One Makefile at the project root. Delegates build/test to `make/` scripts.

```makefile
MAKEFLAGS += --no-print-directory

.PHONY: build test lint fmt clean install uninstall help

BINARY_NAME := project-name

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
- `build` and `test` delegate to `make/` scripts
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

Once `go.mod` is defined, imports within the project:
```go
import "github.com/carlosrabelo/project/internal/processor"
import "github.com/carlosrabelo/project/cmd/main"
```

**Key point**: Import path mirrors the directory structure directly from the module root. Import paths do **not** include `src/`.

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

Define custom error types in `internal/` packages:

```go
// internal/processor/errors.go
package processor

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
// In internal/processor/process.go
func Process(input string) (Result, error) {
    if input == "" {
        return nil, &ValidationError{Field: "input", Message: "empty"}
    }

    data, err := loadData(input)
    if err != nil {
        return nil, fmt.Errorf("load failed: %w", err)
    }

    return data, nil
}

// In cmd/main/main.go
func main() {
    result, err := processor.Process("file.txt")
    if err != nil {
        log.Fatalf("Processing failed: %v", err)
    }

    fmt.Println(result)
}
```

### Anti-Patterns: What NOT to Do

#### Anti-Pattern 1: Putting go.mod in a Subdirectory

```
# BAD: go.mod in src/ (legacy GOPATH-era pattern)
project/
├── Makefile
└── src/
    ├── go.mod          ← Wrong location
    ├── cmd/
    └── internal/

# GOOD: go.mod at project root
project/
├── Makefile
├── go.mod              ← Correct: project root
├── cmd/
└── internal/
```

The `src/` layout is a leftover from the `$GOPATH` era. The official Go documentation (go.dev/doc/modules/layout) explicitly recommends flat root layouts.

#### Anti-Pattern 2: Main Package Logic

```go
// BAD: Logic in main
package main

func main() {
    data, _ := ioutil.ReadFile("config.json")
    // ... 100 more lines of logic
}
```

```go
// GOOD: Extract to internal/ packages
package main

import (
    "log"
    "github.com/carlosrabelo/project/internal/config"
    "github.com/carlosrabelo/project/internal/parser"
)

func main() {
    cfg, err := config.Load("config.json")
    if err != nil {
        log.Fatal(err)
    }

    items, err := parser.ParseItems(cfg)
    if err != nil {
        log.Fatal(err)
    }
}
```

#### Anti-Pattern 3: Ignoring Errors Silently

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

#### Anti-Pattern 4: Generic Package Names

```go
// BAD: Generic names
internal/utils/
internal/helpers/
internal/common/

// GOOD: Specific, domain-driven names
internal/
├── config/      // Configuration loading
├── processor/   // Data processing
├── discovery/   // System discovery
└── validation/  // Input validation
```

#### Anti-Pattern 5: Panicking in Production Code

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

`panic()` is acceptable only in `main.go` for truly unrecoverable scenarios.

#### Anti-Pattern 6: Mixing Concerns in One File

```go
// BAD: Multiple concerns in one file
internal/processor/
└── everything.go  // Config loading, validation, processing, output formatting

// GOOD: Separate by concern
internal/processor/
├── types.go       // Type definitions
├── config.go      // Configuration loading
├── validate.go    // Validation logic
├── process.go     // Core processing
└── output.go      // Result formatting
```

#### Anti-Pattern 7: Not Testing Edge Cases

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

#### Anti-Pattern 8: Two Makefiles

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
├── cmd/
└── internal/
```

#### Anti-Pattern 9: Running `go build` Without `-o`

```bash
# BAD: binary lands in project root
go build ./cmd/project-name

# GOOD: Always use make build or ./make/build.sh
make build
# or
./make/build.sh
```

The `make/build.sh` script always uses `-o "$ROOT_DIR/bin/$BINARY_NAME"`, guaranteeing the binary goes to `bin/`.

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

This skill applies to whichever directory contains `go.mod` — that is the Go project root, regardless of where the git root is.

- `go.mod` lives at `<component>/`, not at the git root
- `<component>/make/` scripts resolve `ROOT_DIR` to the component dir: `$(cd "$(dirname "$0")/.." && pwd)`
- The git root has a separate orchestrator Makefile — this is **not** Anti-Pattern 8 (Two Makefiles)
- When updating `.gitignore`, paths are relative to `<component>/`, not the git root

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

---

## Chaining

When creating a complete Go project from scratch, the full workflow involves these skills in order:

1. **`gitignore-skeleton`** — `.gitignore` with Go patterns (binaries, vendor, caches)
2. **`readme-skeleton`** — `README.md` with the standard structure (handles `README-PT.md` automatically)

After completing a migration:

1. **Check the Makefile** — verify it matches the standard in ## Canonical Layout above (opening lines, .PHONY, help pattern, make/ delegation)
2. **Check the `.gitignore`** — if it is missing the AI Tools section or deviates from the standard, invoke the `gitignore-skeleton` skill
3. **Check the READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill
4. **Commit the changes** — invoke the `git-commit-suggest` skill to stage and commit the reorganization

## Related Skills

- **makefile-skeleton** — Generic Makefile structure conventions (opening lines, .PHONY, help pattern)
- **readme-skeleton** — Standard README content, section order, and bilingual conventions
- **gitignore-skeleton** — For bringing `.gitignore` up to the standard after reorganization
- **monorepo-skeleton** — For organizing multi-language monorepos with Go as one component
