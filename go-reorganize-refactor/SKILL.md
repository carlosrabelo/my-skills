---
name: go-reorganize-refactor
description: Guide to refactoring and reorganizing existing Go code to fit the standard project structure (cmd/, internal/, src/). Handles moving code, creating packages, splitting files, and maintaining tests during refactoring.
mode: agent
category: go
shared: true
---

# Go Code Refactoring and Reorganization

Guide to refactoring existing Go projects to fit the standard project structure pattern. Covers moving code between packages, creating new packages, splitting large files, and maintaining tests during refactoring.

## Overview

This skill helps you:
- Reorganize monolithic Go code into proper packages
- Move code to appropriate locations (`cmd/`, `internal/`)
- Split oversized files into focused modules
- Create new packages with clear responsibilities
- Maintain tests while refactoring
- Update imports after reorganization

---

## Before You Refactor

### Checklist

- ✅ All changes are committed to git (`git commit`)
- ✅ Tests are passing (`make test`)
- ✅ No uncommitted changes (`git status`)
- ✅ You have a backup or clean git history to revert

### Decision: What Needs Refactoring?

Ask yourself:

| Symptom | Problem | Solution |
|---------|---------|----------|
| `main.go` is 300+ lines | Too much logic in `main` | Extract to `internal/` packages |
| Code duplicated across files | Not DRY | Create shared `internal/` package |
| File is 500+ lines | Too big, hard to test | Split into multiple focused files |
| Package name is `utils` or `helpers` | Too generic | Rename to domain-specific name |
| Code used by multiple commands | Duplication | Move to `internal/` |
| Errors ignored with `_` | Poor error handling | Add error handling and types |
| No tests or low coverage | Not testable | Refactor for testability |

---

## Common Refactoring Patterns

### Pattern 1: Extract Logic from main.go to internal/

**Situation**: Your `main.go` has 200+ lines of business logic.

**Before**:
```
src/cmd/myapp/
├── main.go          ← 300 lines (too big!)
└── main_test.go
```

**Steps**:

1. **Create internal package**:
```bash
mkdir -p src/internal/processor
touch src/internal/processor/{types,processor,processor_test,errors}.go
```

2. **Move types to `types.go`**:
```go
// src/internal/processor/types.go
package processor

type Config struct {
    InputFile string
    OutputDir string
}

type Result struct {
    Count    int
    Duration time.Duration
    Errors   []error
}
```

3. **Move business logic**:
```go
// src/internal/processor/processor.go
package processor

func Process(ctx context.Context, cfg Config) (Result, error) {
    // Logic moved from main.go
    result := Result{}

    // actual processing...

    return result, nil
}
```

4. **Add tests**:
```go
// src/internal/processor/processor_test.go
package processor

func TestProcess(t *testing.T) {
    cfg := Config{InputFile: "test.txt", OutputDir: "/tmp"}
    result, err := Process(context.Background(), cfg)

    if err != nil {
        t.Fatalf("Process failed: %v", err)
    }
    if result.Count == 0 {
        t.Error("expected non-zero count")
    }
}
```

5. **Update main.go** (now thin):
```go
// src/cmd/myapp/main.go
package main

import (
    "context"
    "flag"
    "log"
    "github.com/user/project/internal/processor"
)

func main() {
    var inputFile, outputDir string
    flag.StringVar(&inputFile, "input", "", "Input file")
    flag.StringVar(&outputDir, "output", ".", "Output directory")
    flag.Parse()

    cfg := processor.Config{
        InputFile: inputFile,
        OutputDir: outputDir,
    }

    result, err := processor.Process(context.Background(), cfg)
    if err != nil {
        log.Fatal(err)
    }

    log.Printf("Processed: %d items", result.Count)
}
```

6. **Verify and test**:
```bash
cd src
go test ./...      # Tests pass
go build -o ../bin/myapp ./cmd/myapp
../bin/myapp --help # Binary works
```

**After**:
```
src/cmd/myapp/
├── main.go          ← Now ~50 lines
└── main_test.go

src/internal/processor/
├── types.go
├── processor.go
├── processor_test.go
└── errors.go
```

---

### Pattern 2: Extract Duplicated Code into Shared Package

**Situation**: Two commands both have validation logic.

**Before**:
```
src/cmd/
├── command-a/
│   ├── main.go
│   └── validate.go     ← Validation logic
└── command-b/
    ├── main.go
    └── validate.go     ← Same validation logic (copied!)
```

**Steps**:

1. **Create shared package**:
```bash
mkdir -p src/internal/validation
```

2. **Compare the two `validate.go` files**:
```bash
diff src/cmd/command-a/validate.go src/cmd/command-b/validate.go
```

3. **Move common code to shared package** (keep differences minimal):
```go
// src/internal/validation/validator.go
package validation

type Validator struct {
    strictMode bool
}

func NewValidator(strictMode bool) *Validator {
    return &Validator{strictMode: strictMode}
}

func (v *Validator) ValidateInput(input string) error {
    if input == "" {
        return &ValidationError{Field: "input", Message: "empty"}
    }
    // Common validation logic...
    return nil
}
```

4. **Update command-a** to use shared package:
```go
// src/cmd/command-a/main.go
package main

import (
    "github.com/user/project/internal/validation"
)

func main() {
    v := validation.NewValidator(true)
    if err := v.ValidateInput(input); err != nil {
        log.Fatal(err)
    }
    // ...
}
```

5. **Update command-b** similarly:
```go
// src/cmd/command-b/main.go
package main

import (
    "github.com/user/project/internal/validation"
)

func main() {
    v := validation.NewValidator(false)
    if err := v.ValidateInput(input); err != nil {
        log.Fatal(err)
    }
    // ...
}
```

6. **Delete duplicate code**:
```bash
rm src/cmd/command-a/validate.go
rm src/cmd/command-b/validate.go
```

7. **Test**:
```bash
cd src
go test ./...
go build ./cmd/command-a
go build ./cmd/command-b
```

**After**:
```
src/cmd/
├── command-a/
│   └── main.go         ← Uses shared validation
└── command-b/
    └── main.go         ← Uses shared validation

src/internal/validation/
├── types.go
├── validator.go
└── validator_test.go
```

---

### Pattern 3: Split Oversized File

**Situation**: `src/internal/processor/processor.go` is 500 lines.

**Before**:
```
src/internal/processor/
├── processor.go        ← 500 lines (too big!)
├── processor_test.go
└── types.go
```

**Analysis**: Identify the concerns in `processor.go`:
- Loading/parsing input (100 lines)
- Processing logic (200 lines)
- Output formatting (100 lines)
- Error handling (50 lines)
- Helpers (50 lines)

**Steps**:

1. **Create new files** for each concern:
```bash
cd src/internal/processor
mv processor.go processor_main.go  # temporary backup

touch {loader,processor_impl,formatter,errors}.go
```

2. **Extract loader concern**:
```go
// src/internal/processor/loader.go
package processor

func LoadInput(path string) ([]Item, error) {
    // Code from original processor.go
    // ~100 lines
}
```

3. **Extract processing concern**:
```go
// src/internal/processor/processor_impl.go
package processor

func (p *Processor) process(items []Item) Result {
    // Core processing logic
    // ~200 lines
}
```

4. **Extract formatting concern**:
```go
// src/internal/processor/formatter.go
package processor

func FormatOutput(result Result) string {
    // Formatting logic
    // ~100 lines
}
```

5. **Keep types and main interface** in `types.go` and original `processor.go`:
```go
// src/internal/processor/processor.go
package processor

import "context"

// Main public interface
type Processor struct {
    config Config
}

func NewProcessor(cfg Config) *Processor {
    return &Processor{config: cfg}
}

func (p *Processor) Process(ctx context.Context, inputPath string) (Result, error) {
    items, err := LoadInput(inputPath)
    if err != nil {
        return Result{}, fmt.Errorf("load: %w", err)
    }

    result := p.process(items)

    return result, nil
}
```

6. **Update tests**:
```bash
# Original processor_test.go tests public Process() function
# Create new test files for helpers:
touch loader_test.go formatter_test.go
```

7. **Verify**:
```bash
cd src
go test ./internal/processor
go test -v ./internal/processor  # See all tests
wc -l internal/processor/*.go    # Check line counts
```

**After**:
```
src/internal/processor/
├── processor.go        ← ~80 lines (public interface)
├── processor_impl.go   ← ~200 lines (processing)
├── loader.go           ← ~100 lines (loading)
├── formatter.go        ← ~100 lines (formatting)
├── types.go            ← ~50 lines (types)
├── errors.go           ← ~30 lines (error types)
├── processor_test.go   ← Tests for public API
├── loader_test.go      ← Tests for loader
└── formatter_test.go   ← Tests for formatter
```

---

### Pattern 4: Rename Generic Package to Domain-Specific

**Situation**: You have `src/internal/utils/` with utility functions.

**Problem**: Too generic. Functions probably belong to different domains.

**Before**:
```
src/internal/utils/
├── helpers.go   ← String utils, time utils, path utils all mixed
└── helpers_test.go
```

**Steps**:

1. **Analyze what's in utils**:
```bash
grep "^func" src/internal/utils/helpers.go
# Output:
# func TrimWhitespace(s string) string
# func ParseDuration(s string) time.Duration
# func JoinPaths(paths ...string) string
```

2. **Create domain-specific packages**:
```bash
mkdir -p src/internal/stringutil
mkdir -p src/internal/timeutil
mkdir -p src/internal/pathutil
```

3. **Move functions to appropriate packages**:
```go
// src/internal/stringutil/string.go
package stringutil

func TrimWhitespace(s string) string {
    // ...
}

// src/internal/timeutil/time.go
package timeutil

func ParseDuration(s string) time.Duration {
    // ...
}

// src/internal/pathutil/path.go
package pathutil

func JoinPaths(paths ...string) string {
    // ...
}
```

4. **Update all imports** across codebase:
```bash
# Find all files importing from utils
grep -r "internal/utils" src/ --include="*.go"

# Update imports
# Old: import "github.com/user/project/internal/utils"
# New: import "github.com/user/project/internal/stringutil"
```

5. **Delete old package**:
```bash
rm -rf src/internal/utils/
```

6. **Test**:
```bash
cd src
go test ./...
```

**After**:
```
src/internal/
├── stringutil/
│   ├── string.go
│   └── string_test.go
├── timeutil/
│   ├── time.go
│   └── time_test.go
└── pathutil/
    ├── path.go
    └── path_test.go
```

---

### Pattern 5: Create Test Fixtures Directory

**Situation**: Tests have hardcoded data or load files from scattered locations.

**Before**:
```go
// src/internal/processor/processor_test.go
func TestProcess(t *testing.T) {
    data := `{"name":"test","age":30}`  // Hardcoded!
    result := Process(data)
    // ...
}

func TestLoadJSON(t *testing.T) {
    data, _ := ioutil.ReadFile("./test_data.json")  // Wrong path!
    // ...
}
```

**Steps**:

1. **Create testdata directory**:
```bash
mkdir -p src/testdata/processor/{input,expected}
```

2. **Move test data to testdata**:
```bash
# Create test fixtures
echo '{"name":"test","age":30}' > src/testdata/processor/input/valid.json
echo '{"name":"invalid"}' > src/testdata/processor/input/missing_age.json
echo '{"name":"test","age":30,"processed":true}' > src/testdata/processor/expected/result.json
```

3. **Update tests to use testdata**:
```go
// src/internal/processor/processor_test.go
func TestProcess(t *testing.T) {
    data, err := ioutil.ReadFile("testdata/processor/input/valid.json")
    if err != nil {
        t.Fatalf("failed to load fixture: %v", err)
    }

    result := Process(string(data))

    expected, _ := ioutil.ReadFile("testdata/processor/expected/result.json")
    if string(result) != string(expected) {
        t.Errorf("unexpected result")
    }
}
```

4. **Organize testdata logically**:
```
src/testdata/
├── processor/
│   ├── input/
│   │   ├── valid.json
│   │   ├── missing_age.json
│   │   └── invalid_format.json
│   └── expected/
│       ├── result.json
│       └── error.txt
├── loader/
│   ├── input/
│   └── expected/
```

**After**:
```
src/
├── internal/processor/
│   └── processor_test.go    ← Loads from testdata/
└── testdata/
    └── processor/
        ├── input/
        └── expected/
```

---

## Refactoring Checklist

### Before Starting
- [ ] All tests passing
- [ ] All changes committed
- [ ] Clear understanding of what needs moving
- [ ] No uncommitted work

### During Refactoring
- [ ] Move code to new location
- [ ] Update imports in moved code
- [ ] Update imports in code that uses moved code
- [ ] Update test files/imports
- [ ] Run `gofmt` on changed files
- [ ] Tests still pass (`make test`)

### After Refactoring
- [ ] All tests pass
- [ ] Code compiles (`make build`)
- [ ] No unused imports
- [ ] Import paths correct (`go mod tidy`)
- [ ] Binary works as expected
- [ ] Commit with clear message

---

## Useful Commands

### Find all imports of a package
```bash
grep -r "mypackage" src/ --include="*.go"
```

### Find all files with a function
```bash
grep -r "func SomeFunction" src/ --include="*.go"
```

### Check for unused imports
```bash
cd src && go fmt ./...
```

### Check for compilation errors
```bash
cd src && go build ./...
```

### Run tests with verbose output
```bash
cd src && go test -v ./...
```

### Show which files are in a package
```bash
cd src && go list -f '{{.GoFiles}}' ./internal/processor
```

### Move file and update imports intelligently
```bash
# GoLand/VS Code can do this with refactoring tools
# Or manually:
1. Cut file to new location
2. Update package declaration
3. Find all imports of file
4. Update them
5. Run go mod tidy
```

---

## Refactoring Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Moving Code Without Tests

Don't move code without verifying tests still pass:
```bash
# ❌ BAD: Just move the file
mv src/cmd/main/logic.go src/internal/processor/

# ✅ GOOD: Move, update, then test
mv src/cmd/main/logic.go src/internal/processor/
# Update imports in both files
cd src && go test ./...  # Verify tests pass
```

---

### ❌ Anti-Pattern 2: Partial Refactoring

Don't refactor halfway and leave duplicate code:
```
# ❌ BAD: Moved to internal/processor but left copy in cmd/main
src/cmd/main/processor.go      ← Old copy
src/internal/processor/process.go  ← New copy

# ✅ GOOD: Complete refactoring
rm src/cmd/main/processor.go
# Keep only src/internal/processor/
```

---

### ❌ Anti-Pattern 3: Forgetting to Update Imports

After moving code, all imports must be updated:
```go
// ❌ BAD: Old import still used
import "github.com/user/project/cmd/main"

// ✅ GOOD: Updated import
import "github.com/user/project/internal/processor"
```

Run `go build ./...` to catch these.

---

### ❌ Anti-Pattern 4: Mixing Concerns While Refactoring

Don't refactor and change logic at the same time:
```bash
# ❌ BAD: Refactoring + bug fix + feature add
git diff  # 500 lines of changes, mixed concerns

# ✅ GOOD: Refactoring only
git diff  # Only moves and import updates
# Then make functional changes separately
```

---

### ❌ Anti-Pattern 5: Not Running Tests

Don't assume code works after moving:
```bash
# ❌ BAD: Trust and hope
mv src/cmd/main/logic.go src/internal/processor/
git commit

# ✅ GOOD: Verify
cd src && go test ./...  # Pass
cd src && go build ./...  # Compile
# Then commit
```

---

## When to Call Claude with This Skill

Use this skill when you want Claude to help with:
- ✅ Moving code between packages
- ✅ Extracting logic from large files
- ✅ Creating new packages from existing code
- ✅ Splitting oversized files
- ✅ Removing duplication across packages
- ✅ Analyzing which code should move where
- ✅ Updating imports after refactoring

---

## Related Skills

- **go-project-structure** — The standard structure that refactoring should preserve
- **go-commenting-en** — Update comments after moving code between packages
- **git-workflow-go** — Refactoring changes should be separate, atomic commits

---

## Changelog

- **v1.0** (2025-03-12) — Initial version
  - 5 common refactoring patterns with step-by-step guides
  - Before/after examples
  - Refactoring checklist
  - Useful commands for refactoring
  - Anti-patterns to avoid
  - When to use this skill
