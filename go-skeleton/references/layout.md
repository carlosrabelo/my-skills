# Go Project — Standard Layout

Canonical target structure for all Go projects. `go.mod` lives at the project root alongside `cmd/`, `internal/`, and `testdata/`. A single root Makefile handles orchestration, while `make/` scripts do the actual build/test work.

**Key principle**: This structure follows the official Go module layout (go.dev/doc/modules/layout). All Go projects — CLI tools, applications, and libraries — use the same pattern.

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

---

## Core Directories

### `bin/` — Compiled Binaries

**Purpose**: Store executable binaries after compilation.

```bash
bin/project-name              ← Compiled binary
./bin/project-name --help
```

**Note**: Add `bin/` to root `.gitignore` (build artifacts).

---

### `cfg/` — Configuration Files (Optional)

**When to use**:
- ✅ Project requires runtime configuration
- ✅ Need environment-specific configs (dev, staging, prod)
- ❌ Simple CLI with only flags → skip this

```yaml
# cfg/config.yaml
log_level: info
port: 8080
database_url: postgres://localhost/db
timeout: 30
```

---

### `make/` — Automation Scripts

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

---

### `cmd/` — Command Executables

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

---

### `internal/` — Reusable Internal Packages

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

---

## Single Makefile

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

---

## Go Module and Imports

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

---

## .gitignore

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
- `<component>/make/` scripts resolve `ROOT_DIR` to the component dir: `$(cd "$(dirname "$0")/.." && pwd)`
- The git root has a separate orchestrator Makefile — this is **not** Anti-Pattern 8 (Two Makefiles)
