# Go Project — Patterns

Testing patterns, error handling, anti-patterns, and code style guidelines for Go projects.

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
```

- ✅ **80%+** overall coverage
- ✅ **100%** for critical paths
- ❌ Don't obsess over coverage — test behavior, not lines

---

## Error Handling Patterns

### Custom Error Types

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

---

## Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Putting go.mod in a Subdirectory

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

The `src/` layout is a leftover from the `$GOPATH` era. The official Go documentation (go.dev/doc/modules/layout) explicitly recommends flat root layouts.

---

### ❌ Anti-Pattern 2: Main Package Logic

```go
// ❌ BAD: Logic in main
package main

func main() {
    data, _ := ioutil.ReadFile("config.json")
    // ... 100 more lines of logic
}
```

```go
// ✅ GOOD: Extract to internal/ packages
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

---

### ❌ Anti-Pattern 3: Ignoring Errors Silently

```go
// ❌ BAD: Silent failures
data, _ := ioutil.ReadFile("config.json")  // Error ignored!
json.Unmarshal(data, &config)              // May fail

// ✅ GOOD: Handle or propagate errors
data, err := ioutil.ReadFile("config.json")
if err != nil {
    return fmt.Errorf("load config: %w", err)
}

if err := json.Unmarshal(data, &config); err != nil {
    return fmt.Errorf("parse config: %w", err)
}
```

---

### ❌ Anti-Pattern 4: Generic Package Names

```go
// ❌ BAD: Generic names
internal/utils/
internal/helpers/
internal/common/

// ✅ GOOD: Specific, domain-driven names
internal/
├── config/      // Configuration loading
├── processor/   // Data processing
├── discovery/   // System discovery
└── validation/  // Input validation
```

---

### ❌ Anti-Pattern 5: Panicking in Production Code

```go
// ❌ BAD: Panics crash the program
func LoadConfig(file string) Config {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        panic("Config not found!")
    }
}

// ✅ GOOD: Return errors
func LoadConfig(file string) (Config, error) {
    data, err := ioutil.ReadFile(file)
    if err != nil {
        return Config{}, fmt.Errorf("config not found: %w", err)
    }
    return config, nil
}
```

`panic()` is acceptable only in `main.go` for truly unrecoverable scenarios.

---

### ❌ Anti-Pattern 6: Mixing Concerns in One File

```go
// ❌ BAD: Multiple concerns in one file
internal/processor/
└── everything.go  // Config loading, validation, processing, output formatting

// ✅ GOOD: Separate by concern
internal/processor/
├── types.go       // Type definitions
├── config.go      // Configuration loading
├── validate.go    // Validation logic
├── process.go     // Core processing
└── output.go      // Result formatting
```

---

### ❌ Anti-Pattern 7: Not Testing Edge Cases

```go
// ❌ BAD: Only happy path
func TestProcess(t *testing.T) {
    result := Process("valid input")
    if result != "expected" {
        t.Fail()
    }
}

// ✅ GOOD: Table-driven tests with edge cases
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

```
# ❌ BAD: Dual Makefile hierarchy
project/
├── Makefile          ← Root: delegates with make -C src
└── src/
    └── Makefile      ← Second Makefile for Go targets

# ✅ GOOD: Single Makefile at root
project/
├── Makefile          ← Single Makefile handles everything
├── go.mod
├── cmd/
└── internal/
```

---

### ❌ Anti-Pattern 9: Running `go build` Without `-o`

```bash
# ❌ BAD: binary lands in project root
go build ./cmd/project-name

# ✅ GOOD: Always use make build or ./make/build.sh
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
