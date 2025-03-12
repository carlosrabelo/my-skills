---
name: go-project-structure
description: Guide to organizing Go projects with go.mod in src/, dual Makefile hierarchy, cmd/, internal/, and bin/ directories. Covers anti-patterns, testing patterns, error handling, and works for multiple independent Go projects.
mode: agent
category: go
shared: true
---

# Go Project Structure

Comprehensive guide to organizing Go projects following a proven pattern for consistency, maintainability, and scalability across multiple independent projects.

## Overview

This skill explains a pragmatic Go project structure where all Go code lives in `src/` with its own `go.mod`, while the project root handles cross-cutting concerns. Uses a hierarchical Makefile approach: root Makefile for orchestration, `src/Makefile` for Go-specific operations.

**Key principle**: This structure works for CLI tools, applications, and libraries—all your Go projects can follow the same pattern.

---

## Standard Project Layout

```
project-name/
├── Makefile                      ← Root Makefile (orchestrates everything)
│                                   Uses: make -C src target
│
├── bin/                          ← Compiled binaries (from any src build)
│   └── project-name              ← Executable after build
│
├── cfg/                          ← Configuration files (optional)
│   └── config.yaml               ← Default configuration
│
├── run/                          ← Automation scripts
│   ├── build.sh                  ← Build script (cd src && go build)
│   ├── test.sh                   ← Test script
│   ├── clean.sh                  ← Clean script
│   ├── install.sh                ← Install script
│   └── uninstall.sh              ← Uninstall script
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
└── src/                          ← All Go source code
    ├── Makefile                  ← Go-specific targets (build, test, lint)
    │
    ├── go.mod                    ← Go module definition (ALWAYS HERE)
    ├── go.sum                    ← Dependency checksums
    │
    ├── cmd/                      ← Executable entry points
    │   └── project-name/         ← Main command
    │       ├── main.go           ← Entry point
    │       ├── main_test.go      ← Tests
    │       └── *.go              ← Implementation files
    │
    ├── internal/                 ← Internal packages (not importable)
    │   ├── package-one/          ← Reusable logic
    │   │   ├── types.go          ← Type definitions
    │   │   ├── logic.go          ← Core logic
    │   │   ├── logic_test.go     ← Tests
    │   │   └── errors.go         ← Error types
    │   │
    │   └── package-two/          ← Another package
    │       ├── types.go
    │       ├── handler.go
    │       └── handler_test.go
    │
    ├── testdata/                 ← Test fixtures and data
    │   ├── input.json
    │   └── expected.json
    │
    └── .gitignore                ← Go-specific ignores
```

---

## Core Directories

### `bin/` — Compiled Binaries

**Purpose**: Store executable binaries after compilation.

After building, this directory contains ready-to-run executables:
```bash
bin/project-name              ← Compiled binary
```

**Usage**:
```bash
./bin/project-name --help
./bin/project-name command
```

**Note**: Add `bin/` to root `.gitignore` (build artifacts).

---

### `cfg/` — Configuration Files (Optional)

**Purpose**: Store runtime configuration files (YAML, JSON, TOML).

**When to use**:
- ✅ Project requires runtime configuration
- ✅ Need environment-specific configs (dev, staging, prod)
- ❌ Simple CLI with only flags → skip this

**Pattern**:
- Default configuration loaded at startup
- Can be overridden by environment variables
- Support multiple environments

**Example**:
```yaml
# cfg/config.yaml
log_level: info
port: 8080
database_url: postgres://localhost/db
timeout: 30
```

---

### `run/` — Automation Scripts

**Purpose**: Shell scripts for common development and deployment tasks.

**Common scripts**:
```bash
./run/build.sh       # Compile the project (cd src && go build)
./run/test.sh        # Run all tests (cd src && go test)
./run/install.sh     # Install binary to ~/.local/bin
./run/uninstall.sh   # Remove installed binary
```

**Pattern**:
- All scripts are executable (`chmod +x`)
- Named clearly for their purpose
- Execute Go commands directly — do not delegate to make
- Keep scripts simple and focused

**Example `run/build.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR/src"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

---

### `src/` — All Go Code Lives Here

**Purpose**: Container for all Go source code.

**Why separate from root**:
- Keeps Go-specific files organized
- Root can have non-Go files (docs, assets, etc)
- Scales well across multiple independent projects

**Key files**:
- **`go.mod`** — Module definition (ALWAYS in `src/`, never in root)
- **`go.sum`** — Dependency checksums
- **`Makefile`** — Go-specific build targets (see "Makefile Hierarchy" section)
- **`cmd/`, `internal/`, `testdata/`** — See dedicated sections below

---

### `src/cmd/` — Command Executables

**Purpose**: Entry points for CLI commands. Each subdirectory = one executable.

**Structure**:
```
src/cmd/
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

**File organization patterns**:

| File | Purpose |
|------|---------|
| `main.go` | Entry point, parse flags, call functions. Keep ≤50 lines |
| `flags.go` | Command-line flag definitions and parsing |
| `commands.go` | Command routing and coordination logic |
| `types.go` | Type definitions specific to this command |
| `*_test.go` | Tests (same package, one per source file) |

**Example `main.go`**:
```go
package main

import (
    "flag"
    "log"
    "github.com/carlosrabelo/project/internal/processor"
)

func main() {
    var input string
    flag.StringVar(&input, "input", "", "Input file")
    flag.Parse()

    result, err := processor.Process(input)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Result: %v\n", result)
}
```

**Key principle**: Commands contain executable-specific logic. Reusable code goes in `internal/`.

---

### `src/internal/` — Reusable Internal Packages

**Purpose**: Code that's reusable within the project but not importable externally.

**Structure**:
```
src/internal/
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

**Key principle**: Code here is private to the project. External packages cannot import it (Go enforces this).

**When to create a package**:
- ✅ Code is used by multiple commands
- ✅ Logic is complex (100+ lines, distinct domain)
- ✅ Code benefits from being tested separately
- ✅ Clear responsibility and single concern

**When to keep in cmd**:
- ✅ Code is specific to one command
- ✅ Only used by one executable
- ✅ Simple glue code (flag parsing, coordination)

---

### `src/testdata/` — Test Fixtures

**Purpose**: Store test data files (JSON, YAML, CSV, etc).

**Structure**:
```
src/testdata/
├── input/
│   ├── valid.json
│   └── invalid.json
│
└── expected/
    ├── output.json
    └── error.txt
```

**Usage in tests**:
```go
func TestProcess(t *testing.T) {
    data, err := ioutil.ReadFile("testdata/input/valid.json")
    if err != nil {
        t.Fatal(err)
    }

    result := Process(string(data))
    // assertions...
}
```

---

## Makefile Hierarchy

Two Makefiles work in tandem. Keep them **simple and focused** — no unnecessary complexity.

### src/ Makefile

**Purpose**: Go-specific dev tools only. Stays lean: lint, fmt, clean.

**Location**: `./src/Makefile`

**Targets** (only these — nothing else):
```bash
make lint               # Run linter
make fmt                # Format code (gofmt)
make clean              # Remove Go build artifacts
```

**No build/test/install here** — build and test are handled by `run/` scripts, install/uninstall by root Makefile.

**Example src/Makefile**:
```makefile
.PHONY: lint fmt clean

lint:
	go vet ./...

fmt:
	go fmt ./...

clean:
	go clean
```

### Root Makefile

**Purpose**: Single entry point for all operations. Delegates build/test to `run/` scripts, Go dev tools to `src/Makefile`.

**Location**: `./Makefile` (project root)

**Pattern**:
- `build`, `test` → delegate to `./run/build.sh`, `./run/test.sh` (scripts do the real work)
- `install`, `uninstall` → delegate to `./run/install.sh`, `./run/uninstall.sh`
- `lint`, `fmt` → delegate to `make -C src lint/fmt`
- `clean` → `make -C src clean` + `rm -rf bin/`

**Example root Makefile**:
```makefile
MAKEFLAGS += --no-print-directory

.PHONY: build test lint fmt clean install uninstall help

BINARY_NAME := project-name

build:
	./run/build.sh

test:
	./run/test.sh

lint:
	make -C src lint

fmt:
	make -C src fmt

clean:
	make -C src clean
	rm -rf bin/

install: build
	./run/install.sh

uninstall:
	./run/uninstall.sh

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
- `build` and `test` delegate to `run/` scripts — never call `make -C src build` from root
- Use `make -C src` for lint/fmt only
- Keep both Makefiles short and readable

---

## run/ — Shell Scripts

`run/` scripts are an **integral part** of the project structure — not an alternative to Makefiles. They do the actual build/test work directly, while the root Makefile simply invokes them.

**Convention**:
- Scripts execute `cd src && go ...` directly — they do not delegate to `make`
- Always start with `set -euo pipefail`
- Keep scripts focused and executable (`chmod +x`)
- Name clearly by purpose

**Standard scripts**:
```bash
run/
├── build.sh       # Build binary to ../bin/
├── test.sh        # Run all tests
├── install.sh     # Install to ~/.local/bin
└── uninstall.sh   # Remove from ~/.local/bin
```

**Example `run/build.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

mkdir -p "$ROOT_DIR/bin"
cd "$ROOT_DIR/src"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

**Example `run/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cd "$ROOT_DIR/src"
go test -v ./...
```

**Example `run/install.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/bin/$BINARY_NAME" "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

**Example `run/uninstall.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"

rm -f "$INSTALL_DIR/$BINARY_NAME"
echo "Removed: $INSTALL_DIR/$BINARY_NAME"
```

---

## Go Module and Imports

### Module Definition

**Location**: `src/go.mod`

Your `go.mod` file defines the module path. Always use the GitHub-based pattern:

**Standard (required)**
```
module github.com/carlosrabelo/project-name
```

**Exception: Simple path (for throwaway/internal-only projects)**
```
module project-name
```

### Import Paths

Once `go.mod` is defined, imports within your project work like this:

**If `go.mod` says `module github.com/carlosrabelo/project`:**
```go
// In any Go file
import "github.com/carlosrabelo/project/internal/processor"
import "github.com/carlosrabelo/project/cmd/main"

// NOT "github.com/carlosrabelo/project/src/internal/processor"
// The src/ directory is invisible to imports
```

**Example**:
```go
// src/cmd/main/main.go
package main

import (
    "log"
    "github.com/carlosrabelo/project/internal/processor"
)

func main() {
    result := processor.Process("data")
    log.Println(result)
}
```

**Key point**: The import path **does not include `src/`** even though code is physically in `src/`. Go determines the module root from `go.mod` location.

---

## File Organization Patterns

### Naming Files by Function

Name files by what they do, not generic names:

**Good**:
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

**Avoid**:
```go
// ❌ Too generic
// utils.go
// helpers.go
// common.go
```

### Test Files

Tests live in the same package and directory as source:

```
src/internal/processor/
├── process.go          ← Implementation
├── process_test.go     ← Tests for process.go
├── types.go            ← Type definitions
├── types_test.go       ← Tests for types (if complex)
└── errors.go           ← Error types

src/cmd/main/
├── main.go             ← Entry point
├── main_test.go        ← Integration tests
└── flags.go            ← Flag parsing
```

### Types Organization

**Option 1: Centralized (for small packages)**
```
src/internal/config/
├── types.go            ← All types here
├── load.go             ← Config loading logic
└── load_test.go
```

**Option 2: Distributed (for larger packages)**
```
src/internal/processor/
├── types.go            ← Core types
├── process.go          ← Type Processor + logic
├── process_test.go
├── validator.go        ← Type Validator + logic
└── validator_test.go
```

---

## Testing Patterns

### Table-Driven Tests

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

### Test Fixtures

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

### Coverage Targets

```bash
# Run with coverage
make -C src test-coverage

# View HTML report
open src/coverage.html
```

Aim for:
- ✅ **80%+** overall coverage
- ✅ **100%** for critical paths
- ❌ Don't obsess over coverage—test behavior, not lines

---

## Error Handling Patterns

### Error Types

Define custom error types in `internal/` packages:

```go
// src/internal/processor/errors.go
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

### Error Wrapping

Use `fmt.Errorf("%w", err)` to wrap errors:

```go
// ✅ Good: preserves error chain
result, err := loadConfig("config.yaml")
if err != nil {
    return fmt.Errorf("failed to load configuration: %w", err)
}

// ❌ Bad: loses error context
if err != nil {
    return err  // Caller loses context
}

// ❌ Bad: doesn't preserve error type
if err != nil {
    return fmt.Errorf("error: %v", err)  // Can't use errors.Is()
}
```

### Error Checking

```go
// ✅ Check specific error types
if errors.Is(err, os.ErrNotExist) {
    // handle missing file
}

// ✅ Check custom types
var ve *ValidationError
if errors.As(err, &ve) {
    log.Printf("Validation failed on field: %s", ve.Field)
}
```

### When to Log vs Return

**Return error if**:
- Caller should decide what to do
- Error can be recovered at higher level
- Error affects program logic

**Log error if**:
- Unrecoverable situation
- Informational logging
- Error is already being returned (don't double-log)

**Example**:
```go
// In internal/processor/process.go
func Process(input string) (Result, error) {
    // Return error: caller might handle it
    if input == "" {
        return nil, &ValidationError{Field: "input", Message: "empty"}
    }

    // Return error: caller can retry or fallback
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
        // Log: this is the top level, log and exit
        log.Fatalf("Processing failed: %v", err)
    }

    fmt.Println(result)
}
```

---

## Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Main Package Logic

Don't put business logic in `main.go`:

```go
// ❌ BAD: Logic in main
package main

func main() {
    data, _ := ioutil.ReadFile("config.json")
    var config Config
    json.Unmarshal(data, &config)

    items := []Item{}
    for _, line := range strings.Split(string(data), "\n") {
        items = append(items, parseItem(line))
    }
    // ... 100 more lines of logic
}
```

**✅ GOOD: Extract to internal/ packages**

```go
// cmd/main/main.go
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

    processItems(items)
}
```

---

### ❌ Anti-Pattern 2: Ignoring Errors Silently

Never ignore errors without good reason:

```go
// ❌ BAD: Silent failures
data, _ := ioutil.ReadFile("config.json")  // Error ignored!
json.Unmarshal(data, &config)              // May fail
processor.Process(config)                   // Undefined behavior
```

**✅ GOOD: Handle or propagate errors**

```go
data, err := ioutil.ReadFile("config.json")
if err != nil {
    return fmt.Errorf("load config: %w", err)
}

if err := json.Unmarshal(data, &config); err != nil {
    return fmt.Errorf("parse config: %w", err)
}

if err := processor.Process(config); err != nil {
    return fmt.Errorf("process: %w", err)
}
```

---

### ❌ Anti-Pattern 3: Generic Package Names

Avoid `utils/`, `helpers/`, `common/`:

```go
// ❌ BAD: Generic names
src/internal/utils/
src/internal/helpers/
src/internal/common/
```

**✅ GOOD: Specific, domain-driven names**

```go
src/internal/
├── config/      // Configuration loading
├── processor/   // Data processing
├── discovery/   // System discovery
└── validation/  // Input validation
```

---

### ❌ Anti-Pattern 4: Panicking in Production Code

Never use `panic()` in libraries or internal packages:

```go
// ❌ BAD: Panics crash the program
func LoadConfig(file string) Config {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        panic("Config not found!")  // Crashes entire program!
    }
    // ...
}
```

**✅ GOOD: Return errors**

```go
func LoadConfig(file string) (Config, error) {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        return Config{}, fmt.Errorf("config not found: %w", err)
    }
    // ...
    return config, nil
}
```

Exception: `panic()` is acceptable only in `main.go` for truly unrecoverable scenarios.

---

### ❌ Anti-Pattern 5: Mixing Concerns in One File

Keep files focused on one responsibility:

```go
// ❌ BAD: Multiple concerns in one file
src/internal/processor/
└── everything.go  // Config loading, validation, processing, output formatting
```

**✅ GOOD: Separate by concern**

```go
src/internal/processor/
├── types.go       // Type definitions
├── config.go      // Configuration loading
├── validate.go    // Validation logic
├── process.go     // Core processing
└── output.go      // Result formatting
```

---

### ❌ Anti-Pattern 6: Not Testing Edge Cases

Don't just test the happy path:

```go
// ❌ BAD: Only happy path
func TestProcess(t *testing.T) {
    result := Process("valid input")
    if result != "expected" {
        t.Fail()
    }
}
```

**✅ GOOD: Test edge cases with table-driven tests**

```go
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

---

## Key Principles

### 1. Clear Separation of Concerns

- **`cmd/`** = Executable-specific logic and CLI interface
- **`internal/`** = Reusable, testable business logic
- **`cfg/`** = Runtime configuration defaults (optional)
- **`run/`** = Development automation scripts
- **`bin/`** = Compiled outputs (don't commit)
- **`testdata/`** = Test fixtures and data

### 2. Package Naming

Package name matches directory name (without `src/` in the name):

```
src/internal/setup/    → package setup
src/internal/config/   → package config
src/cmd/main/          → package main
```

### 3. Testing

```
file.go              → Implementation
file_test.go         → Tests (same package)
testdata/            → Test fixtures
```

Run with `make test` or `go test ./...`

### 4. Documentation

- **README.md** — English documentation
- **README-PT.md** — Portuguese documentation
- Document the project and how to build it

### 5. Visibility

- **`cmd/`** — Private to the executable
- **`internal/`** — Private to the project (Go enforces this)
- **`bin/`** — Build outputs (add to `.gitignore`)
- **`pkg/`** — Public (only if creating a library)

### 6. Go Module Location

- **`src/go.mod`** — Always here, never in root
- **`src/go.sum`** — Always here, never in root
- Root `Makefile` calls `make -C src` for Go operations

---

## Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add new CLI command | `src/cmd/command-name/` | One binary per directory |
| Add reusable logic | `src/internal/package-name/` | Used by multiple commands |
| Add default configuration | `cfg/config.yaml` | Only if config needed |
| Add build script | `run/script-name.sh` | Delegates to Makefiles |
| Add test fixtures | `src/testdata/` | JSON, YAML, CSV, etc |
| Add library for external use | `src/pkg/library-name/` | Only for publishable libs |

---

## Example: Adding a New Feature

**Scenario**: Add file validation capability to existing project

```
Step 1: Create internal package
  mkdir -p src/internal/validator

Step 2: Create files
  src/internal/validator/types.go           ← Type definitions
  src/internal/validator/validate.go        ← Validation logic
  src/internal/validator/validate_test.go   ← Tests
  src/internal/validator/errors.go          ← Error types

Step 3: Update command to use it
  src/cmd/main/main.go
  import "github.com/carlosrabelo/project/internal/validator"

  // In main():
  if err := validator.ValidateFile(inputFile); err != nil {
      log.Fatal(err)
  }

Step 4: Add tests
  cd src && make test

Step 5: Verify structure
  cd src && tree -I 'vendor|go.sum' -L 3
```

---

## When to Create a New Package

### Create a package if:
✅ Code is used by multiple commands
✅ Logic is complex (100+ lines, distinct domain)
✅ Code benefits from being tested separately
✅ Clear, single responsibility

### Keep in cmd if:
✅ Code is specific to one command
✅ Only used by one executable
✅ Simple glue code (flag parsing, coordination)

---

## Best Practices

### Organizing cmd/ packages
- Keep `main.go` ≤50 lines
- Put flag parsing in `flags.go`
- Put core logic in `*_impl.go` or domain-specific files
- Use `internal/` for reusable code
- Write integration tests in `main_test.go`

### Organizing internal/ packages
- One responsibility per package
- Group related types together in `types.go`
- Keep test coverage high (80%+)
- Document exported functions and types
- Use error types from `errors.go`

### File size guidelines
- `main.go` ≤ 50 lines
- Other files ≤ 200 lines (split if larger)
- Package = one clear concern
- Function ≤ 50 lines (guideline, not rule)

### Code quality
- Run `gofmt` before commits
- Use linters (golangci-lint)
- Write table-driven tests
- Document exported functions (godoc)
- Keep error chains intact with `%w`

---

## Example Project Structure

### Simple CLI Tool (Makalu-like)

```
makalu/
├── Makefile
├── bin/
├── cfg/                  (optional)
├── run/
│   ├── build.sh
│   └── test.sh
├── README.md
└── src/
    ├── Makefile
    ├── go.mod
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
        ├── diff/
        │   ├── types.go
        │   ├── compare.go
        │   └── compare_test.go
        └── suggestion/
            ├── types.go
            ├── suggest.go
            └── suggest_test.go
```

### Larger Application

```
app/
├── Makefile
├── bin/
├── cfg/
│   └── config.yaml
├── run/
│   ├── build.sh
│   ├── test.sh
│   └── deploy.sh
├── LICENSE
├── README.md
├── README-PT.md
└── src/
    ├── Makefile
    ├── go.mod
    ├── go.sum
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
    │   ├── storage/
    │   │   ├── db.go
    │   │   ├── db_test.go
    │   │   ├── queries.go
    │   │   └── types.go
    │   └── services/
    │       ├── processor.go
    │       ├── processor_test.go
    │       ├── validator.go
    │       └── validator_test.go
    └── testdata/
        ├── input/
        │   ├── valid.json
        │   └── invalid.json
        └── expected/
            ├── output.json
            └── error.txt
```

---

## Workflows

### Building
```bash
# From project root
make build

# Or directly in src
cd src && make build
```

### Testing
```bash
# From project root
make test

# Or directly in src
cd src && make test
cd src && make test-coverage
```

### Installing
```bash
# From project root (builds then installs)
make install

# Remove
make uninstall
```

### Development iteration
```bash
# From project root (standard workflow)
make fmt                     # Format code
make lint                    # Run linter
make test                    # Run tests
make build                   # Build binary
./bin/project-name --help    # Test it

# Quick Go dev loop (inside src/ directly)
cd src
go fmt ./...
go vet ./...
go test ./...
```

---

## .gitignore

Add to root `.gitignore`:
```
# Build artifacts
bin/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Go
src/coverage.out
src/coverage.html
```

The `src/` directory should be committed (including `go.mod` and `go.sum`).

---

## Related Skills

- **go-reorganize-refactor** — For restructuring code within this pattern
- **go-commenting-en** — For consistent English comments throughout
- **git-workflow-go** — For atomic commits and clean history in Go projects
