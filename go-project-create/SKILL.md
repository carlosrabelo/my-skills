---
name: go-project-create
description: Guide to organizing Go projects with go.mod at project root, single Makefile, cmd/, internal/, and bin/ directories. Covers anti-patterns, testing patterns, error handling, and works for multiple independent Go projects.
mode: agent
category: go
shared: true
---

# Go Project Structure

Comprehensive guide to organizing Go projects following modern Go conventions for consistency, maintainability, and scalability across multiple independent projects.

## Overview

This skill explains a flat root Go project structure where `go.mod` lives at the project root alongside `cmd/`, `internal/`, and `testdata/`. A single root Makefile handles orchestration, while `run/` scripts do the actual build/test work.

**Key principle**: This structure follows the official Go module layout (go.dev/doc/modules/layout). All your Go projects — CLI tools, applications, and libraries — use the same pattern.

---

## Standard Project Layout

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
├── run/                          ← Automation scripts
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
./run/build.sh       # Compile the project (go build)
./run/test.sh        # Run all tests (go test)
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
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

---

### `cmd/` — Command Executables

**Purpose**: Entry points for CLI commands. Each subdirectory = one executable.

**Structure**:
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

### `internal/` — Reusable Internal Packages

**Purpose**: Code that's reusable within the project but not importable externally.

**Structure**:
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

### `testdata/` — Test Fixtures

**Purpose**: Store test data files (JSON, YAML, CSV, etc).

**Structure**:
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

## Single Makefile

One Makefile at the project root. Keep it **simple and focused** — no unnecessary complexity.

**Purpose**: Single entry point for all operations. Delegates build/test to `run/` scripts.

**Location**: `./Makefile` (project root)

**Pattern**:
- `build`, `test` → delegate to `./run/build.sh`, `./run/test.sh` (scripts do the real work)
- `install`, `uninstall` → delegate to `./run/install.sh`, `./run/uninstall.sh`
- `lint`, `fmt`, `clean` → run Go commands directly

**Example Makefile**:
```makefile
MAKEFLAGS += --no-print-directory

.PHONY: build test lint fmt clean install uninstall help

BINARY_NAME := project-name

build:
	./run/build.sh

test:
	./run/test.sh

lint:
	go vet ./...

fmt:
	go fmt ./...

clean:
	go clean
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
- `build` and `test` delegate to `run/` scripts
- `lint`, `fmt`, `clean` run Go commands directly (no second Makefile needed)
- Keep it short and readable

---

## run/ — Shell Scripts

`run/` scripts are an **integral part** of the project structure — not an alternative to the Makefile. They do the actual build/test work, while the root Makefile simply invokes them.

**Convention**:
- Scripts execute `cd "$ROOT_DIR" && go ...` directly — they do not delegate to `make`
- Always start with `set -euo pipefail`
- Keep scripts focused and executable (`chmod +x`)
- Name clearly by purpose

**Standard scripts**:
```bash
run/
├── build.sh       # Build binary to bin/
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
cd "$ROOT_DIR"
go build -o "$ROOT_DIR/bin/$BINARY_NAME" "./cmd/$BINARY_NAME"
echo "Binary ready at: $ROOT_DIR/bin/$BINARY_NAME"
```

**Example `run/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cd "$ROOT_DIR"
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

**Location**: `go.mod` (project root)

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
```

**Example**:
```go
// cmd/main/main.go
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

**Key point**: The import path mirrors the directory structure directly from the module root.

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
internal/processor/
├── process.go          ← Implementation
├── process_test.go     ← Tests for process.go
├── types.go            ← Type definitions
├── types_test.go       ← Tests for types (if complex)
└── errors.go           ← Error types

cmd/main/
├── main.go             ← Entry point
├── main_test.go        ← Integration tests
└── flags.go            ← Flag parsing
```

### Types Organization

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
go test -coverprofile=coverage.out ./...

# View HTML report
go tool cover -html=coverage.out -o coverage.html
open coverage.html
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

### ❌ Anti-Pattern 1: Putting go.mod in a Subdirectory

Don't nest `go.mod` inside `src/` or any other subdirectory:

```
# ❌ BAD: go.mod in src/ (legacy GOPATH-era pattern)
project/
├── Makefile
└── src/
    ├── go.mod          ← Wrong location
    ├── cmd/
    └── internal/

# ✅ GOOD: go.mod at project root
project/
├── Makefile
├── go.mod              ← Correct: project root
├── cmd/
└── internal/
```

The `src/` layout is a leftover from the `$GOPATH` era. Modern Go (with modules) expects `go.mod` at the project root. The official Go documentation (go.dev/doc/modules/layout) explicitly recommends flat root layouts.

---

### ❌ Anti-Pattern 2: Main Package Logic

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

### ❌ Anti-Pattern 3: Ignoring Errors Silently

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

### ❌ Anti-Pattern 4: Generic Package Names

Avoid `utils/`, `helpers/`, `common/`:

```go
// ❌ BAD: Generic names
internal/utils/
internal/helpers/
internal/common/
```

**✅ GOOD: Specific, domain-driven names**

```go
internal/
├── config/      // Configuration loading
├── processor/   // Data processing
├── discovery/   // System discovery
└── validation/  // Input validation
```

---

### ❌ Anti-Pattern 5: Panicking in Production Code

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

### ❌ Anti-Pattern 6: Mixing Concerns in One File

Keep files focused on one responsibility:

```go
// ❌ BAD: Multiple concerns in one file
internal/processor/
└── everything.go  // Config loading, validation, processing, output formatting
```

**✅ GOOD: Separate by concern**

```go
internal/processor/
├── types.go       // Type definitions
├── config.go      // Configuration loading
├── validate.go    // Validation logic
├── process.go     // Core processing
└── output.go      // Result formatting
```

---

### ❌ Anti-Pattern 7: Not Testing Edge Cases

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

### ❌ Anti-Pattern 8: Two Makefiles

Don't use a second Makefile in a subdirectory:

```
# ❌ BAD: Dual Makefile hierarchy
project/
├── Makefile          ← Root: delegates with make -C src
└── src/
    └── Makefile      ← Second Makefile for Go targets
```

**✅ GOOD: Single Makefile at root**

```
project/
├── Makefile          ← Single Makefile handles everything
├── go.mod
├── cmd/
└── internal/
```

---

### ❌ Anti-Pattern 9: Running `go build` Without `-o`

Running `go build` directly without the `-o` flag places the binary in the current working directory, not in `bin/`:

```bash
# ❌ BAD: binary lands in project root
go build ./cmd/project-name

# ❌ BAD: also lands in project root
cd cmd/project-name && go build
```

**✅ GOOD: Always use `make build` or `./run/build.sh`**

```bash
# ✅ Uses run/build.sh which specifies -o bin/project-name
make build

# ✅ Or call the script directly
./run/build.sh
```

The `run/build.sh` script always uses `-o "$ROOT_DIR/bin/$BINARY_NAME"`, which guarantees the binary goes to `bin/` regardless of the working directory.

As a safety net, add the binary name to `.gitignore` so a misplaced binary is never accidentally committed:

```gitignore
# Build artifacts
bin/
project-name          ← binary name without path, catches root-level builds
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

Package name matches directory name:

```
internal/setup/    → package setup
internal/config/   → package config
cmd/main/          → package main
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

- **`go.mod`** — Always at project root
- **`go.sum`** — Always at project root
- Single root Makefile for all operations

---

## Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add new CLI command | `cmd/command-name/` | One binary per directory |
| Add reusable logic | `internal/package-name/` | Used by multiple commands |
| Add default configuration | `cfg/config.yaml` | Only if config needed |
| Add build script | `run/script-name.sh` | Delegates to Go commands |
| Add test fixtures | `testdata/` | JSON, YAML, CSV, etc |
| Add library for external use | `pkg/library-name/` | Only for publishable libs |

---

## Example: Adding a New Feature

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

### Simple CLI Tool

```
makalu/
├── Makefile
├── go.mod
├── bin/
├── cfg/                  (optional)
├── run/
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
├── go.mod
├── go.sum
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

# Or directly
go build -o bin/project-name ./cmd/project-name
```

### Testing
```bash
# From project root
make test

# Or directly
go test -v ./...
go test -coverprofile=coverage.out ./...
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
```

---

## .gitignore

Add to root `.gitignore`:
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

## Monorepo Usage

This skill applies to whichever directory contains `go.mod` — that is the Go project root, regardless of where the git root is.

- `go.mod` lives at `<component>/`, not at the git root
- `<component>/run/` scripts resolve `ROOT_DIR` to the component dir: `$(cd "$(dirname "$0")/.." && pwd)`
- The git root has a separate orchestrator Makefile — this is **not** Anti-Pattern 8 (Two Makefiles)

See **monorepo-project-create** for the full monorepo layout, root Makefile patterns, and component naming conventions.

---

## Code Comments

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

## Encadeamento

Ao criar um projeto Go completo do zero, o fluxo completo envolve estas skills na ordem:

1. **`makefile-create`** — criar o Makefile com os targets padrão
2. **`readme-create`** — criar `README.md` com a estrutura padrão
3. **`readme-bilingual-sync`** — criar `README-PT.md` como tradução de `README.md`

Estas skills são automáticas (modo `agent`) e são acionadas naturalmente ao criar os respectivos arquivos. A ordem acima é a sequência correta.

---

## Related Skills

- **go-project-migrate** — For restructuring code within this pattern (invoke manually with `/go-project-migrate`)
- **makefile-create** — Standard Makefile structure and targets used in every Go project
- **readme-create** — Standard README content, section order, and bilingual conventions
- **monorepo-project-create** — For organizing multi-language monorepos with Go as one component
