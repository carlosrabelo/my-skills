---
name: go-commenting-en
description: Guide to writing clear, idiomatic English comments and documentation in Go code. Covers comment style, doc comments for godoc, inline comments, and documentation best practices.
mode: agent
category: go
shared: true
---

# Go Commenting and Documentation

Guide to writing clear, consistent, and idiomatic English comments and documentation in Go code. Follows Go's conventions and best practices for code clarity.

## Overview

Good comments in Go:
- ✅ Are written in clear English (clear and concise)
- ✅ Explain **why**, not just **what** (code shows what)
- ✅ Follow godoc conventions for documentation
- ✅ Use consistent style across the codebase
- ✅ Are updated when code changes

---

## Comment Types in Go

### 1. Package Comments

**Purpose**: Describe the entire package and its purpose.

**Location**: Above the `package` declaration in any `.go` file (conventionally in a `doc.go` file or at the top of the main file).

**Format**: Single-line or multi-line comment block.

**Examples**:

```go
// Package processor provides data processing capabilities.
//
// It handles loading, validating, and transforming input files.
// Processing results can be formatted in multiple output formats.
package processor
```

Or for more complex packages:

```go
/*
Package validator provides input validation for the project.

It includes:
  - Basic type validation
  - Custom business logic validation
  - Error types for validation failures

Example usage:

	v := validator.NewValidator()
	if err := v.ValidateInput(input); err != nil {
		// Handle validation error
	}

For more details, see the documentation in the package.
*/
package validator
```

**Rules**:
- First line should be package name + brief description
- Can include multiple sentences or paragraphs
- Should be complete sentences ending with periods
- Use blank lines to separate logical sections

---

### 2. Exported Function/Type Comments

**Purpose**: Document exported functions, types, and their behavior.

**Location**: Immediately above the declaration.

**Format**: Single-line comment starting with name of the exported identifier.

**Examples**:

```go
// Process reads input from the given path and returns the result.
//
// It returns an error if the file doesn't exist or parsing fails.
func Process(path string) (Result, error) {
    // ...
}

// Config holds configuration for the processor.
type Config struct {
    InputFile  string        // Path to input file
    OutputDir  string        // Directory for output
    Timeout    time.Duration // Processing timeout
}

// NewConfig creates a new Config with default values.
func NewConfig() *Config {
    return &Config{
        OutputDir: ".",
        Timeout:   30 * time.Second,
    }
}
```

**Rules**:
- Start with the identifier name (function/type)
- Complete sentence ending with period
- If multiple lines, first line is summary, then blank line, then details
- For exported types, also document exported fields

---

### 3. Unexported (Internal) Function Comments

**Purpose**: Document internal functions for package maintainability.

**Location**: Immediately above the declaration.

**Format**: Comment starting with function name (lowercase).

**Examples**:

```go
// validateInput checks if the input string meets requirements.
// It returns nil if valid, or a ValidationError otherwise.
func validateInput(input string) error {
    // ...
}

// processLine handles a single line of input.
// It assumes the line has already been validated.
func processLine(line string) (Item, error) {
    // ...
}
```

**Rules**:
- Same format as exported functions
- Starting with lowercase (function name as written)
- Still use complete sentences
- Explain non-obvious logic or assumptions

---

### 4. Inline Comments

**Purpose**: Explain complex or non-obvious code.

**Location**: Above or at the end of the code line.

**Format**: `// Comment explaining this line/block`

**Examples**:

```go
// ✅ GOOD: Explains non-obvious logic
func calculateScore(items []Item) int {
    score := 0

    // Weight recent items higher than old items
    for i, item := range items {
        weight := 1.0 + float64(i)*0.1
        score += int(float64(item.Value) * weight)
    }

    return score
}

// ❌ BAD: Explains obvious code
func add(a, b int) int {
    sum := a + b  // Add a to b
    return sum    // Return the sum
}
```

**Rules**:
- Only for complex, non-obvious code
- Explain **why**, not **what**
- Complete sentences preferred, but short phrases acceptable
- Don't state the obvious

---

### 5. TODO and FIXME Comments

**Purpose**: Mark known issues, improvements, or incomplete work.

**Location**: Inline, above problematic code.

**Format**: `// TODO: Description` or `// FIXME: Description`

**Examples**:

```go
// TODO: Add support for CSV format output
func formatOutput(data []Item, format string) string {
    switch format {
    case "json":
        return formatJSON(data)
    case "csv":
        // TODO: Implement CSV formatting
        return ""
    default:
        return ""
    }
}

// FIXME: This panics if input is nil, should return error instead
func Process(input *Data) Result {
    if input == nil {
        panic("input cannot be nil")  // FIXME: Return error
    }
    // ...
}
```

**Rules**:
- Be specific about what needs to be done
- Consider adding a context (why it's needed)
- Clean up before releasing major versions
- Useful for marking technical debt

---

### 6. Deprecated Comments

**Purpose**: Mark functions or types that should no longer be used.

**Location**: Immediately above the declaration.

**Format**: `// Deprecated: Explanation of why and what to use instead.`

**Examples**:

```go
// Deprecated: Use Process instead. This function will be removed in v2.0.
func OldProcess(path string) Result {
    return Process(path)
}

// Deprecated: LoadConfig is deprecated. Use config.Load instead.
func LoadConfig(file string) (Config, error) {
    return config.Load(file)
}
```

**Rules**:
- Clearly state the reason for deprecation
- Suggest the replacement
- Mention version when it will be removed
- Still maintain the function (don't delete it)

---

## DocComment Conventions (Godoc)

Godoc automatically generates documentation from comments. Follow these conventions:

### General Rules

1. **Capitalization**: Start with uppercase letter
2. **Punctuation**: End with period
3. **Blank lines**: Separate paragraphs with blank lines
4. **Code blocks**: Indent code examples 1 tab (continues from normal text)
5. **Lists**: Use bullet points with hyphens

### Example with all conventions:

```go
// Package database provides database operations.
//
// It supports multiple databases (PostgreSQL, MySQL, SQLite)
// and provides a common interface for queries and transactions.
//
// Basic usage:
//
//	db, err := database.Open("postgres", "connection_string")
//	if err != nil {
//		log.Fatal(err)
//	}
//	defer db.Close()
//
//	results, err := db.Query("SELECT * FROM users")
//	if err != nil {
//		log.Fatal(err)
//	}
//
// Database supports the following drivers:
//  - PostgreSQL (driver: "postgres")
//  - MySQL (driver: "mysql")
//  - SQLite (driver: "sqlite")
//
// For more information, see the individual package documentation.
package database

// Query performs a SQL query and returns results.
//
// It handles connection pooling automatically. The query runs
// in the context of the database connection and may fail if the
// connection is closed.
//
// Query is not goroutine-safe for a single connection. Use separate
// connections for concurrent queries.
//
// Example:
//
//	rows, err := db.Query("SELECT id, name FROM users WHERE id = ?", userID)
//	if err != nil {
//		// Handle error
//	}
//	defer rows.Close()
//	for rows.Next() {
//		// Process row
//	}
func (db *Database) Query(query string, args ...interface{}) (*Rows, error) {
    // ...
}

// User represents a user in the database.
type User struct {
    ID    int       // Unique identifier
    Name  string    // User's full name
    Email string    // User's email address
    Active bool     // Whether user is active
}
```

---

## Comment Structure for Complex Functions

For more complex functions, use a structured comment:

```go
// ValidateAndProcess validates input and processes it.
//
// Validation checks include:
//  - File exists and is readable
//  - File is valid JSON
//  - JSON contains required fields
//
// Processing includes:
//  - Transforming data to internal format
//  - Caching results
//  - Recording metrics
//
// It returns an error if validation fails at any step.
// Processing errors are logged but not returned (best effort).
func ValidateAndProcess(path string) (Result, error) {
    // ...
}
```

---

## Comments for Different Scenarios

### Error Handling

```go
// ✅ GOOD: Explains why error is expected or unrecoverable
data, err := ioutil.ReadFile(path)
if err != nil {
    // File not found is expected for optional config files
    if errors.Is(err, os.ErrNotExist) {
        return defaultConfig, nil
    }
    // Other errors are unexpected and unrecoverable
    return nil, fmt.Errorf("read config: %w", err)
}

// ❌ BAD: States the obvious
if err != nil {  // If error is not nil
    return err   // Return error
}
```

### Business Logic

```go
// ✅ GOOD: Explains the business rule
func discountForLoyalty(purchaseAmount float64, membershipYears int) float64 {
    // Loyal customers (3+ years) get 10% discount, others get 5%
    if membershipYears >= 3 {
        return purchaseAmount * 0.9
    }
    return purchaseAmount * 0.95
}

// ❌ BAD: Explains code, not logic
func discountForLoyalty(purchaseAmount float64, membershipYears int) float64 {
    if membershipYears >= 3 {  // If membership years is 3 or more
        return purchaseAmount * 0.9  // Return 90% of amount
    }
    return purchaseAmount * 0.95  // Return 95% of amount
}
```

### Algorithm Explanation

```go
// ✅ GOOD: Explains algorithm and complexity
// quickSort uses the Hoare partition scheme.
// Average case: O(n log n)
// Worst case: O(n²) when pivot is always smallest/largest
// In practice, it's faster than merge sort due to cache locality.
func quickSort(arr []int) {
    // Implementation...
}

// ❌ BAD: Comments every line
func quickSort(arr []int) {
    if len(arr) <= 1 {  // Base case
        return
    }
    pivot := arr[0]     // Choose first as pivot
    i := 1              // Start from second element
    // ... etc
}
```

---

## Field Comments in Structs

For exported struct fields, add comments:

```go
// ✅ GOOD: Field comments explain purpose and units
type Config struct {
    // InputFile is the path to the input file.
    InputFile string

    // Timeout is the maximum time to process a file.
    // Zero means no timeout.
    Timeout time.Duration

    // RetryCount is the number of times to retry on failure.
    // Must be non-negative.
    RetryCount int

    // Verbose enables detailed logging output.
    Verbose bool
}

// ❌ BAD: No field comments
type Config struct {
    InputFile  string
    Timeout    time.Duration
    RetryCount int
    Verbose    bool
}

// ❌ BAD: Obvious comments
type Config struct {
    InputFile  string        // The input file
    Timeout    time.Duration // The timeout
    RetryCount int           // The retry count
    Verbose    bool          // Is it verbose
}
```

---

## Comments for Assumptions and Constraints

Mark important assumptions clearly:

```go
// ✅ GOOD: Assumptions are explicit
func processData(items []Item) Result {
    // Assumes items are already sorted by date (ascending)
    // If unsorted, results will be incorrect

    // Assumes no item's value exceeds MaxInt64
    var total int64
    for _, item := range items {
        total += int64(item.Value)
    }

    return total
}

// ✅ GOOD: Constraints are documented
func newPool(size int) (*Pool, error) {
    // Size must be between 1 and 1000
    if size < 1 || size > 1000 {
        return nil, fmt.Errorf("size must be 1-1000, got %d", size)
    }
    // ...
}
```

---

## What NOT to Comment

### ❌ Don't Comment Obvious Code

```go
// BAD
x := x + 1  // Increment x
if len(arr) == 0 {  // If array is empty
    return  // Return
}

// GOOD: No comments needed - code is clear
x++
if len(arr) == 0 {
    return
}
```

### ❌ Don't Comment Variable Names

```go
// BAD
userID := 123  // The user ID
userName := "John"  // The user name

// GOOD: Name itself is clear
userID := 123
userName := "John"
```

### ❌ Don't Duplicate Type Information

```go
// BAD
func ListUsers() []User {  // Returns a slice of User structs
    // ...
}

// GOOD: Document intent, not type
func ListUsers() []User {
    // Returns all active users from the database.
    // ...
}
```

### ❌ Don't Comment Syntax

```go
// BAD
if x > 0 {  // Open brace for if statement
    y = x * 2  // Multiply x by 2
}  // Close brace

// GOOD: No syntax comments needed
if x > 0 {
    y = x * 2
}
```

---

## Comment Maintenance

### When to Update Comments

- ✅ When code logic changes
- ✅ When behavior changes
- ✅ When adding new features
- ✅ When fixing bugs

### Review Process

Before committing:

```bash
git diff  # Review all changes
# For each commented change, ask:
# - Is the comment still accurate?
# - Is the comment still necessary?
# - Is the comment clear?
```

### Stale Comments

Mark outdated comments:

```go
// ⚠️ STALE COMMENT (remove in next refactor):
// This function is slow and should be optimized
// See: https://github.com/user/project/issues/123
func slowFunction() {
    // ...
}
```

---

## Commenting Conventions Summary

| Type | Example | Rules |
|------|---------|-------|
| Package | `// Package X provides...` | Start with package name |
| Exported function | `// FuncName does X.` | Start with function name |
| Unexported function | `// funcName does X.` | Start with function name (lowercase) |
| Struct field | `// Field describes X` | Complete sentence |
| Inline | `// Explain why here` | Only for non-obvious code |
| TODO | `// TODO: Description` | Be specific |
| Deprecated | `// Deprecated: Use X instead.` | Suggest replacement |

---

## Examples by Package Type

### CLI Tool Package

```go
// Package commands provides CLI command handlers.
//
// Each command in the CLI tool is implemented as a separate function
// that processes flags and input, then calls the appropriate business logic
// from the internal packages.
//
// Example:
//
//	cmd, err := NewProcessCommand(flags)
//	if err != nil {
//		log.Fatal(err)
//	}
//	result, err := cmd.Execute()
package commands

// ProcessCommand handles the "process" CLI command.
//
// It validates input files, runs processing, and formats output
// according to the flags provided.
type ProcessCommand struct {
    InputFile  string
    OutputDir  string
    Verbose    bool
}

// Execute runs the process command.
//
// It returns an error if input validation fails or processing encounters
// an unrecoverable error. Non-critical errors are logged but not returned.
func (c *ProcessCommand) Execute(ctx context.Context) (Result, error) {
    // ...
}
```

### Data Processing Package

```go
// Package processor provides data processing operations.
//
// The package handles three main operations:
//  - Loading raw data from files
//  - Validating data against business rules
//  - Transforming data to the desired output format
package processor

// Process loads, validates, and transforms the given input file.
//
// It returns an error if the file cannot be read, validation fails,
// or any unrecoverable error occurs during processing.
//
// Processing is best-effort: partial results are not returned on error.
func Process(ctx context.Context, inputPath string) (Result, error) {
    // ...
}
```

### Configuration Package

```go
// Package config provides configuration loading and validation.
package config

// Load reads configuration from the given file path.
//
// Supported formats: JSON, YAML, TOML
//
// Environment variables override config file values.
// Format: APP_<SECTION>_<KEY> maps to config.Section.Key
//
// Example:
//
//	cfg, err := Load("config.yaml")
//	if err != nil {
//		log.Fatal(err)
//	}
//	// Environment variable APP_DATABASE_URL overrides cfg.Database.URL
func Load(path string) (Config, error) {
    // ...
}
```

---

## Common Mistakes and Fixes

### Mistake 1: Over-Commenting

```go
// ❌ BAD: Too many obvious comments
func Add(a, b int) int {
    // Add a to b
    sum := a + b
    // Return sum
    return sum
}

// ✅ GOOD: One-line comment if needed at all
// Add returns the sum of a and b.
func Add(a, b int) int {
    return a + b
}
```

### Mistake 2: Comments Getting Out of Sync

```go
// ❌ BAD: Comment doesn't match code
func Process(items []Item) Result {
    // Returns only the first item
    return result  // Actually returns all items!
}

// ✅ GOOD: Comment matches code
func Process(items []Item) Result {
    // Returns all items that passed validation
    return result
}
```

### Mistake 3: Comments That Add No Value

```go
// ❌ BAD: Comment just repeats code
data, err := loadFile(path)
if err != nil {
    return err  // Error occurred
}

// ✅ GOOD: Comment explains why error handling works this way
data, err := loadFile(path)
if err != nil {
    return err  // Already logged by loadFile, no need to log again
}
```

### Mistake 4: Comments in Wrong Language or Grammar

```go
// ❌ BAD: Poor English or typos
func Validate(input string) bool {
    // Check the input is valid data
    // Returne true if ok
    return check(input)
}

// ✅ GOOD: Clear, grammatically correct English
func Validate(input string) bool {
    // Returns true if input is valid, false otherwise.
    return check(input)
}
```

---

## Tools for Commenting

### Godoc Generation

```bash
cd src
# Generate HTML documentation from comments
go doc ./internal/processor

# View full documentation
godoc -http=:6060
# Then open http://localhost:6060/pkg/github.com/user/project
```

### Linters for Comments

```bash
# Using golangci-lint (includes comment checks)
cd src && golangci-lint run ./...
```

### IDE Highlighting

Most IDEs (VS Code, GoLand) have comment linting built in and will warn you about:
- Exported functions without doc comments
- Malformed doc comments
- Inconsistent formatting

---

## Checklist for Good Comments

Before committing:

- [ ] All exported functions have doc comments
- [ ] All exported types have doc comments
- [ ] Exported struct fields have comments
- [ ] Complex logic has explanatory comments
- [ ] TODO/FIXME comments are specific
- [ ] No comments state the obvious
- [ ] No comments that duplicate code
- [ ] All comments are in English
- [ ] All comments are grammatically correct
- [ ] Comments match the code they describe

---

## Related Skills

- **go-project-structure** — Context for where different comment types appear (cmd/, internal/)
- **go-reorganize-refactor** — Update comments when moving code between packages

---

## Changelog

- **v1.0** (2025-03-12) — Initial version
  - Package, function, and type comment conventions
  - Inline and TODO comments
  - Godoc best practices
  - Field and struct comments
  - Comments for business logic and algorithms
  - Common mistakes and fixes
  - Checklist for good comments
