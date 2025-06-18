# Go Project — Creating from Scratch

Instructions for creating a new Go project following the standard layout. Read `layout.md` for the canonical structure before applying these instructions.

## File Organization Patterns

### Naming Files by Function

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

### Test Files

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

## Key Principles

### 1. Clear Separation of Concerns

- **`cmd/`** = Executable-specific logic and CLI interface
- **`internal/`** = Reusable, testable business logic
- **`cfg/`** = Runtime configuration defaults (optional)
- **`make/`** = Development automation scripts
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
| Add build script | `make/script-name.sh` | Delegates to Go commands |
| Add test fixtures | `testdata/` | JSON, YAML, CSV, etc |
| Add library for external use | `pkg/library-name/` | Only for publishable libs |

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

## Example Project Structures

### Simple CLI Tool

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

### Larger Application

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

---

## Best Practices

### Organizing cmd/ packages
- Keep `main.go` ≤50 lines
- Put flag parsing in `flags.go`
- Put core logic in domain-specific files
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

## Development Workflows

```bash
# From project root
make build

# Or directly
go build -o bin/project-name ./cmd/project-name
```

```bash
make test

# Or directly
go test -v ./...
go test -coverprofile=coverage.out ./...
```

```bash
# Build then install
make install

# Remove
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
