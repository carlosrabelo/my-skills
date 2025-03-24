---
name: python-reorganize-refactor
description: Guide to refactoring and reorganizing existing Python code to fit the standard project structure (flat root layout, sub-packages only when truly needed, tests/). Handles moving modules, creating packages, splitting files, and maintaining tests during refactoring. Invoke manually with /python-reorganize-refactor.
mode: manual
category: python
shared: true
---

# Python Code Refactoring and Reorganization

Guide to refactoring existing Python projects to fit the standard project structure pattern. Covers moving code between modules, creating sub-packages only when genuinely needed, splitting large files, and maintaining tests during refactoring.

## Overview

This skill helps you:
- Reorganize monolithic Python code into the flat root layout (source files at project root)
- Move code to appropriate locations (domain-specific files at project root)
- Create sub-packages only when multiple files form a cohesive module
- Split oversized files into focused modules
- Maintain tests while refactoring
- Update imports after reorganization

Invoke manually with `/python-reorganize-refactor`, or when `python-project-structure` suggests it.

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
| `main.py` is 200+ lines | Too much logic in the entry point | Extract to domain-specific modules at project root |
| Code duplicated across files | Not DRY | Create shared module at project root |
| File is 400+ lines | Too big, hard to test | Split into multiple focused files |
| Module named `utils.py` or `helpers.py` | Too generic | Rename to domain-specific name |
| Code used by multiple modules | Duplication | Move to shared module at project root |
| Bare `except:` or ignored errors | Poor error handling | Add custom exception types in `errors.py` |
| No tests or low coverage | Not testable | Refactor for testability |
| Sub-package with only one file | Unnecessary packaging | Flatten to `module.py` at root |
| Files inside `src/` directory | Old layout | Move to project root, update shebag and config |

---

## Common Refactoring Patterns

### Pattern 1: Extract Logic from main.py to Module

**Situation**: Your `main.py` has 200+ lines of business logic.

**Before**:
```
project-name/
├── main.py          ← 250 lines (too big!)
└── errors.py
```

**Steps**:

1. **Create processor module**:
```bash
touch processor.py
touch tests/test_processor.py
```

2. **Define errors in `errors.py`** (if not already):
```python
# errors.py
"""Custom exception types."""


class ProjectError(Exception):
    """Base exception for all project errors."""


class ValidationError(ProjectError):
    """Raised when input validation fails."""

    def __init__(self, field: str, message: str) -> None:
        self.field = field
        self.message = message
        super().__init__(f"validation failed on {field}: {message}")


class ProcessingError(ProjectError):
    """Raised when processing fails."""
```

3. **Move business logic to `processor.py`**:
```python
# processor.py
"""Core processing logic."""

from pathlib import Path

from errors import ProcessingError, ValidationError


def process(input_path: str) -> str:
    """Process the given input and return the result."""
    if not input_path:
        raise ValidationError("input_path", "cannot be empty")

    path = Path(input_path)
    if not path.exists():
        raise ValidationError("input_path", f"file not found: {input_path}")

    try:
        return path.read_text(encoding="utf-8")
    except OSError as e:
        raise ProcessingError(f"failed to read file: {e}") from e
```

4. **Add tests**:
```python
# tests/test_processor.py
"""Tests for processor."""

import pytest
from pathlib import Path

from processor import process
from errors import ValidationError


class TestProcess:
    def test_valid_input(self, tmp_path: Path) -> None:
        f = tmp_path / "input.txt"
        f.write_text("hello", encoding="utf-8")
        assert process(str(f)) == "hello"

    def test_empty_path(self) -> None:
        with pytest.raises(ValidationError) as exc_info:
            process("")
        assert exc_info.value.field == "input_path"

    def test_missing_file(self) -> None:
        with pytest.raises(ValidationError):
            process("/nonexistent/path.txt")
```

5. **Update `main.py`** (now thin):
```python
#!/usr/bin/env python3
"""Command-line interface."""

import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent / ".venv" / "bin" / "python"
if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

import argparse

from processor import process
from errors import ProcessingError, ValidationError


def main() -> None:
    parser = argparse.ArgumentParser(description="What the tool does")
    parser.add_argument("input", help="Input file path")
    args = parser.parse_args()

    try:
        result = process(args.input)
        print(result)
    except ValidationError as e:
        print(f"Bad input: {e}", file=sys.stderr)
        sys.exit(1)
    except ProcessingError as e:
        print(f"Processing failed: {e}", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

6. **Verify and test**:
```bash
make test
make lint
make typecheck
```

**After**:
```
project-name/
├── main.py          ← ~30 lines (entry point only)
├── processor.py     ← business logic
└── errors.py        ← custom exceptions

tests/
├── test_main.py
└── test_processor.py
```

---

### Pattern 2: Extract Duplicated Code into Shared Module

**Situation**: Two scripts both have the same validation logic.

**Before**:
```
project-name/
├── convert.py    ← Has validate_path() function
└── export.py     ← Same validate_path() function (copied!)
```

**Steps**:

1. **Create shared module**:
```bash
touch validation.py
touch tests/test_validation.py
```

2. **Compare the two implementations**:
```bash
grep -A 10 "def validate_path" convert.py
grep -A 10 "def validate_path" export.py
```

3. **Move common code to shared module**:
```python
# validation.py
"""Shared input validation."""

from pathlib import Path

from errors import ValidationError


def validate_path(path: str, *, must_exist: bool = True) -> Path:
    """Validate a file path argument."""
    if not path:
        raise ValidationError("path", "cannot be empty")
    p = Path(path)
    if must_exist and not p.exists():
        raise ValidationError("path", f"file not found: {path}")
    return p
```

4. **Update `convert.py`**:
```python
# convert.py
from validation import validate_path

def run(input_path: str) -> None:
    path = validate_path(input_path)
    # ...
```

5. **Update `export.py`** similarly:
```python
# export.py
from validation import validate_path

def run(input_path: str, output_path: str) -> None:
    src = validate_path(input_path)
    dst = validate_path(output_path, must_exist=False)
    # ...
```

6. **Delete duplicate code** from both files and run tests:
```bash
make test
```

**After**:
```
project-name/
├── convert.py      ← Uses shared validation
├── export.py       ← Uses shared validation
└── validation.py   ← Single source of truth
```

---

### Pattern 3: Split Oversized File

**Situation**: `processor.py` is 450 lines.

**Before**:
```
project-name/
├── processor.py    ← 450 lines (too big!)
└── errors.py
```

**Analysis**: Identify the concerns in `processor.py`:
- Loading/parsing input (100 lines)
- Processing logic (200 lines)
- Output formatting (100 lines)

**Steps**:

1. **Create new files** for each concern:
```bash
touch loader.py formatter.py
touch tests/test_loader.py tests/test_formatter.py
```

2. **Extract loader concern**:
```python
# loader.py
"""Input loading and parsing."""

from pathlib import Path

from errors import ProcessingError


def load_input(path: str) -> list[str]:
    """Load and parse input file, returning lines."""
    try:
        return Path(path).read_text(encoding="utf-8").splitlines()
    except OSError as e:
        raise ProcessingError(f"failed to load {path}: {e}") from e
```

3. **Extract formatting concern**:
```python
# formatter.py
"""Output formatting."""


def format_result(items: list[str]) -> str:
    """Format processed items for output."""
    return "\n".join(items)
```

4. **Keep the main interface** in `processor.py` (now thin):
```python
# processor.py
"""Core processing logic."""

from loader import load_input
from formatter import format_result
from errors import ProcessingError, ValidationError


def process(input_path: str) -> str:
    """Process the given input file and return formatted result."""
    if not input_path:
        raise ValidationError("input_path", "cannot be empty")

    items = load_input(input_path)
    processed = [item.strip() for item in items if item.strip()]
    return format_result(processed)
```

5. **Verify**:
```bash
make test
make lint
wc -l *.py
```

**After**:
```
project-name/
├── processor.py    ← ~20 lines (public interface)
├── loader.py       ← ~20 lines (loading)
├── formatter.py    ← ~15 lines (formatting)
└── errors.py       ← ~20 lines (error types)

tests/
├── test_processor.py
├── test_loader.py
└── test_formatter.py
```

---

### Pattern 4: Rename Generic Module to Domain-Specific

**Situation**: You have `utils.py` with mixed utility functions.

**Problem**: Too generic. Functions belong to different domains.

**Before**:
```
project-name/
└── utils.py   ← string helpers, path helpers, time helpers mixed
```

**Steps**:

1. **Analyze what's in utils.py**:
```bash
grep "^def " utils.py
# Output:
# def trim_whitespace(s: str) -> str:
# def parse_duration(s: str) -> float:
# def join_paths(*paths: str) -> str:
```

2. **Create domain-specific modules**:
```bash
touch text.py
touch time_utils.py
touch paths.py
```

3. **Move functions to appropriate modules**:
```python
# text.py
"""Text processing utilities."""


def trim_whitespace(s: str) -> str:
    return s.strip()


# time_utils.py
"""Time parsing utilities."""


def parse_duration(s: str) -> float:
    ...


# paths.py
"""Path utilities."""

from pathlib import Path


def join_paths(*paths: str) -> Path:
    return Path(*paths)
```

4. **Update all imports** across the codebase:
```bash
# Find all files importing from utils
grep -r "from utils import" *.py tests/
grep -r "import utils" *.py tests/

# Update imports in each file
# Old: from utils import trim_whitespace
# New: from text import trim_whitespace
```

5. **Delete old module**:
```bash
rm utils.py
```

6. **Test**:
```bash
make test
make typecheck
```

**After**:
```
project-name/
├── text.py
├── time_utils.py
└── paths.py
```

---

### Pattern 5: Flatten Unnecessary Sub-Package

**Situation**: A sub-package has only one file and no real reason to be a package.

**Before**:
```
project-name/
└── models/
    ├── __init__.py
    └── item.py       ← only one file in this package
```

**Steps**:

1. **Move the file up**:
```bash
mv models/item.py item.py
rm models/__init__.py
rmdir models/
```

2. **Update all imports**:
```bash
# Find all files importing from models
grep -r "from models" *.py tests/
grep -r "from models.item" *.py tests/

# Old: from models.item import Item
# New: from item import Item
```

3. **Update tests path**:
```bash
mv tests/models/test_item.py tests/test_item.py
rmdir tests/models/
```

4. **Verify**:
```bash
make test
```

**After**:
```
project-name/
└── item.py    ← flat, no unnecessary package wrapper
```

---

### Pattern 6: Introduce Sub-Package When Truly Needed

**Situation**: You have 5+ model files that should be grouped.

**Before**:
```
project-name/
├── user.py
├── item.py
├── order.py
├── product.py
└── invoice.py    ← 5 model-type files, hard to distinguish from logic files
```

**Steps**:

1. **Create the sub-package**:
```bash
mkdir -p models
touch models/__init__.py
```

2. **Move the files**:
```bash
mv user.py models/
mv item.py models/
mv order.py models/
mv product.py models/
mv invoice.py models/
```

3. **Update imports**:
```bash
# Old: from user import User
# New: from models.user import User
```

4. **Move tests**:
```bash
mkdir -p tests/models
touch tests/models/__init__.py
mv tests/test_user.py tests/models/
# etc.
```

5. **Verify**:
```bash
make test
```

**After**:
```
project-name/
└── models/         ← justified: 5 cohesive model files
    ├── __init__.py
    ├── user.py
    ├── item.py
    ├── order.py
    ├── product.py
    └── invoice.py
```

---

### Pattern 7: Create Proper Test Fixtures with conftest.py

**Situation**: Tests have hardcoded data or scattered temporary files.

**Before**:
```python
# tests/test_processor.py
def test_process():
    content = "line1\nline2\nline3"  # Hardcoded!
    with open("/tmp/test_input.txt", "w") as f:  # Hardcoded path!
        f.write(content)
    result = process("/tmp/test_input.txt")
    assert result == "line1\nline2\nline3"
```

**Steps**:

1. **Create shared fixtures in `conftest.py`**:
```python
# tests/conftest.py
"""Shared pytest fixtures."""

import pytest
from pathlib import Path


@pytest.fixture
def sample_text_file(tmp_path: Path) -> Path:
    """Create a temporary text file for testing."""
    f = tmp_path / "input.txt"
    f.write_text("line1\nline2\nline3", encoding="utf-8")
    return f


@pytest.fixture
def empty_file(tmp_path: Path) -> Path:
    """Create an empty temporary file."""
    f = tmp_path / "empty.txt"
    f.write_text("", encoding="utf-8")
    return f
```

2. **Create `tests/fixtures/` for static test data**:
```bash
mkdir -p tests/fixtures
echo '{"name": "test", "value": 42}' > tests/fixtures/valid.json
echo '{"name": "invalid"' > tests/fixtures/malformed.json
```

3. **Add a fixture to load static files**:
```python
# tests/conftest.py (continued)
FIXTURES_DIR = Path(__file__).parent / "fixtures"


@pytest.fixture
def fixtures_dir() -> Path:
    """Return path to the fixtures directory."""
    return FIXTURES_DIR
```

4. **Update tests to use fixtures**:
```python
# tests/test_processor.py
from pathlib import Path
from processor import process


class TestProcess:
    def test_valid_input(self, sample_text_file: Path) -> None:
        result = process(str(sample_text_file))
        assert "line1" in result

    def test_json_input(self, fixtures_dir: Path) -> None:
        result = process(str(fixtures_dir / "valid.json"))
        assert result is not None
```

**After**:
```
tests/
├── conftest.py         ← Shared fixtures using tmp_path
├── fixtures/           ← Static test data files
│   ├── valid.json
│   └── malformed.json
└── test_processor.py
```

---

### Pattern 8: Migrate from src/ Layout to Root Layout

**Situation**: Project uses the old `src/` layout and needs to move to flat root.

**Before**:
```
project-name/
├── src/
│   ├── fetcher.py
│   ├── processor.py
│   └── errors.py
├── tests/
├── pyproject.toml
└── Makefile
```

**Steps**:

1. **Move source files to project root**:
```bash
mv src/*.py .
# Move sub-packages if any
# mv src/models/ .
rmdir src/
```

2. **Update shebag in entry point** — change `parent.parent` to `parent`:
```python
# Old (src/ layout):
_venv_python = Path(__file__).resolve().parent.parent / ".venv" / "bin" / "python"

# New (root layout):
_venv_python = Path(__file__).resolve().parent / ".venv" / "bin" / "python"
```

3. **Update `pyproject.toml`**:
```toml
# Old:
# [tool.ruff]
# src = ["src"]
# [tool.pytest.ini_options]
# pythonpath = ["src"]

# New:
[tool.ruff]
src = ["."]

[tool.pytest.ini_options]
pythonpath = ["."]
```

4. **Update `Makefile`** — replace `src/` references:
```makefile
# Old:
# fmt:
# 	.venv/bin/ruff format src/ tests/
# typecheck:
# 	.venv/bin/mypy src/

# New:
PY_FILES := $(wildcard *.py)

fmt:
	.venv/bin/ruff format $(PY_FILES) tests/

typecheck:
	.venv/bin/mypy $(PY_FILES)
```

5. **Update `run/lint.sh`**:
```bash
# Old: "$ROOT_DIR/.venv/bin/ruff" check src/ tests/
# New:
"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR"/*.py tests/
```

6. **Update `run/install.sh`**:
```bash
# Old: cp "$ROOT_DIR/src/$SCRIPT_NAME" "$INSTALL_DIR/$BINARY_NAME"
# New:
cp "$ROOT_DIR/$SCRIPT_NAME" "$INSTALL_DIR/$BINARY_NAME"
```

7. **Remove `sys.path` hacks from tests** (no longer needed with `pythonpath = ["."]`):
```python
# Delete these lines from test files:
# import sys
# import os
# sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "src"))
```

8. **Verify**:
```bash
make test
make lint
make typecheck
```

**After**:
```
project-name/
├── fetcher.py
├── processor.py
├── errors.py
├── tests/
├── pyproject.toml
└── Makefile
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
- [ ] Add/update `__init__.py` only for new sub-packages
- [ ] Update test files and imports
- [ ] Tests still pass (`make test`)

### After Refactoring
- [ ] All tests pass (`make test`)
- [ ] Type checks pass (`make typecheck`)
- [ ] Linter passes (`make lint`)
- [ ] No unused imports (`ruff check`)
- [ ] Commit with clear message

---

## Useful Commands

### Find all imports of a module
```bash
grep -r "from utils import" *.py tests/
grep -r "import utils" *.py tests/
```

### Find all definitions in a file
```bash
grep "^def \|^class " utils.py
```

### Check for unused imports
```bash
.venv/bin/ruff check *.py tests/
```

### Check for type errors
```bash
.venv/bin/mypy *.py
```

### Run tests with coverage
```bash
.venv/bin/pytest --cov=. --cov-report=term-missing
```

### Run a specific test file
```bash
.venv/bin/pytest tests/test_processor.py -v
```

---

## Refactoring Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Moving Code Without Tests

```bash
# ❌ BAD: Just move the file
mv logic.py processor.py

# ✅ GOOD: Move, update imports, then verify
mv logic.py processor.py
# Update all imports referencing the old name
make test  # Verify tests pass
```

---

### ❌ Anti-Pattern 2: Partial Refactoring

```
# ❌ BAD: Moved logic but left original file
processor.py      ← Old copy (still referenced!)
new_processor.py  ← New copy

# ✅ GOOD: Complete the refactoring
rm processor.py
# Keep only new_processor.py with updated imports everywhere
```

---

### ❌ Anti-Pattern 3: Forgetting to Update Imports

```python
# ❌ BAD: Old import still used
from utils import trim_whitespace

# ✅ GOOD: Updated import
from text import trim_whitespace
```

Run `make lint` and `make typecheck` to catch these.

---

### ❌ Anti-Pattern 4: Mixing Concerns While Refactoring

```bash
# ❌ BAD: Refactoring + bug fix + new feature all at once
git diff  # 400 lines of mixed changes

# ✅ GOOD: Refactoring only in one commit
git diff  # Only moves and import updates
# Then make functional changes in separate commits
```

---

### ❌ Anti-Pattern 5: Not Running Tests

```bash
# ❌ BAD: Trust and hope
mv core.py processor.py
git commit

# ✅ GOOD: Verify at each step
make test       # Tests pass
make typecheck  # No type errors
make lint       # No lint errors
git commit
```

---

### ❌ Anti-Pattern 6: Missing `__init__.py` in Sub-Packages

```bash
# ❌ BAD: New sub-package directory without __init__.py
mkdir models/
# Forgot: touch models/__init__.py

# ✅ GOOD: Always create __init__.py for new sub-packages
mkdir models/
touch models/__init__.py
```

---

### ❌ Anti-Pattern 7: Creating Sub-Packages for a Single File

```
# ❌ BAD: Sub-package wrapping a single file
validation/
├── __init__.py
└── rules.py    ← only one file, no reason to be a package

# ✅ GOOD: Single file directly at project root
validation.py
```

---

## When to Call This Skill

Invoke manually with `/python-reorganize-refactor` when you want help with:
- ✅ Moving code between modules
- ✅ Extracting logic from large files
- ✅ Creating sub-packages from existing code (only when justified)
- ✅ Flattening unnecessary sub-packages
- ✅ Splitting oversized files
- ✅ Removing duplication across modules
- ✅ Analyzing which code should move where
- ✅ Updating imports after reorganization
- ✅ Migrating from `src/` layout to flat root layout

---

## Related Skills

- **python-project-structure** — The standard structure that refactoring should target
- **git-workflow-go** — Refactoring changes should be separate, atomic commits (applies equally to Python)
- **readme-bilingual-sync** — Update docs after significant structural changes
