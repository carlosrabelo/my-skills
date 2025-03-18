---
name: python-project-structure
description: Guide to organizing Python projects with pyproject.toml, .venv, src/ layout, Makefile hierarchy, and run/ scripts. Main files live directly in src/, sub-packages only when truly needed. Covers shebag using ../.venv, testing patterns, error handling.
mode: agent
category: python
shared: true
---

# Python Project Structure

Comprehensive guide to organizing Python projects following a consistent pattern for maintainability and scalability across multiple independent projects.

## Overview

This skill explains a pragmatic Python project structure where the main source files live directly in `src/`, the virtual environment stays at the project root as `.venv`, and `pyproject.toml` defines the project. Uses a hierarchical Makefile approach: root Makefile for orchestration, `run/` scripts for the actual work.

**Key principles**:
- `.venv` is always at the project root — never inside `src/`
- `pyproject.toml` is the single source of truth for metadata and dependencies
- Main scripts live directly in `src/` — sub-packages only when a true module is needed
- Scripts in `src/` use `../.venv` (one level up) for the self-relaunching shebag
- Python version: **3.12.3**

---

## Standard Project Layout

```
project-name/
├── Makefile                      ← Root Makefile (orchestrates everything)
│
├── .venv/                        ← Virtual environment (NEVER commit)
│
├── pyproject.toml                ← Project metadata, dependencies, tool config
│
├── run/                          ← Automation scripts
│   ├── setup.sh                  ← Create .venv and install dependencies
│   ├── test.sh                   ← Run tests
│   ├── lint.sh                   ← Run linter
│   └── install.sh                ← Install script(s) to ~/.local/bin
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
├── src/                          ← All Python source files
│   ├── main.py                   ← Main entry point (executable, has shebag)
│   ├── processor.py              ← Core logic module
│   └── errors.py                 ← Custom exception types
│   └── models/                   ← Sub-package ONLY if there are multiple related modules
│       ├── __init__.py
│       └── item.py
│
└── tests/                        ← All tests (outside src/)
    ├── conftest.py               ← Pytest fixtures and configuration
    ├── test_main.py
    ├── test_processor.py
    └── models/                   ← Mirror sub-packages from src/ if they exist
        └── test_item.py
```

### When to Create a Sub-Package in src/

Only create a sub-folder inside `src/` when:
- It groups **multiple files** that form a cohesive module (e.g., `models/`, `api/`)
- It needs an `__init__.py` to be importable as a package
- A single file would become too large or have mixed concerns

**Never** create `utils/`, `helpers/`, or `common/` folders — give the module a domain-specific name.

---

## Core Files

### `pyproject.toml` — Project Definition

**Location**: Project root (never inside `src/`).

**Purpose**: Single file for metadata, dependencies, build system, and tool configuration.

**Example** (script-based project, no installable package):
```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "project-name"
version = "0.1.0"
description = "Short description of what the project does"
readme = "README.md"
license = { file = "LICENSE" }
requires-python = ">=3.12"
dependencies = [
    # Add runtime dependencies here
    # "requests>=2.31",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov>=5.0",
    "ruff>=0.4",
    "mypy>=1.10",
]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"

[tool.ruff]
src = ["src"]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

[tool.mypy]
python_version = "3.12"
strict = true
```

If the project has an installable package (sub-package in `src/`), add:
```toml
[tool.setuptools.packages.find]
where = ["src"]

[project.scripts]
project-name = "package_name.cli:main"
```

---

### `.venv/` — Virtual Environment

**Location**: Always at project root.

**Create with**:
```bash
python3 -m venv .venv
```

**Activate**:
```bash
source .venv/bin/activate
```

**Install dependencies**:
```bash
pip install -e ".[dev]"
# or, for script-only projects without a package:
pip install ".[dev]"
```

**Never commit** — add to `.gitignore`.

---

## Makefile

**Purpose**: Single entry point for all operations. Delegates to `run/` scripts.

**Location**: Project root.

```makefile
MAKEFLAGS += --no-print-directory

.PHONY: setup test lint fmt typecheck clean install help

setup:
	./run/setup.sh

test:
	./run/test.sh

lint:
	./run/lint.sh

fmt:
	.venv/bin/ruff format src/ tests/

typecheck:
	.venv/bin/mypy src/

clean:
	rm -rf dist/ build/ *.egg-info src/*.egg-info
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete

install: setup
	./run/install.sh

help:
	@echo "Usage: make <target>"
	@echo ""
	@echo "Targets:"
	@echo "  setup      Create .venv and install dependencies"
	@echo "  test       Run tests"
	@echo "  lint       Run linter (ruff)"
	@echo "  fmt        Format code (ruff format)"
	@echo "  typecheck  Run type checker (mypy)"
	@echo "  clean      Remove build artifacts and __pycache__"
	@echo "  install    Install script(s) to ~/.local/bin"
```

---

## run/ — Shell Scripts

Scripts do the actual work directly. The Makefile simply invokes them.

**Convention**:
- Always start with `set -euo pipefail`
- Activate `.venv` before running Python commands
- Keep scripts simple and focused

**Example `run/setup.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

if [ ! -d "$ROOT_DIR/.venv" ]; then
    echo "Creating virtual environment..."
    python3 -m venv "$ROOT_DIR/.venv"
fi

echo "Installing dependencies..."
"$ROOT_DIR/.venv/bin/pip" install -e "$ROOT_DIR[dev]" --quiet
echo "Setup complete."
```

**Example `run/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

"$ROOT_DIR/.venv/bin/pytest" "$ROOT_DIR/tests/" "$@"
```

**Example `run/lint.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

"$ROOT_DIR/.venv/bin/ruff" check src/ tests/
```

**Example `run/install.sh`**:
```bash
#!/bin/bash
set -euo pipefail

SCRIPT_NAME="main.py"
BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/src/$SCRIPT_NAME" "$INSTALL_DIR/$BINARY_NAME"
chmod +x "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

---

## Shebags and .venv

Python scripts in `src/` meant to be run directly (e.g. `./src/main.py`) must point to the right interpreter. On Linux, shebags **must be absolute paths** — relative paths like `#!../.venv/bin/python` are not supported by the kernel.

The solution is the **self-relaunching pattern**: the script starts with the system Python, detects `.venv` at the project root (`..` relative to `src/`), and re-executes itself using the `.venv` interpreter if not already running inside it.

### The Self-Relaunching Pattern

Scripts live in `src/`, so `.venv` is one level up (`parent.parent`):

```python
#!/usr/bin/env python3
"""Script description."""

# Auto-switch to ../.venv (project root is one level above src/)
import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent.parent / ".venv" / "bin" / "python"

if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# --- script starts here, guaranteed to be running inside .venv ---

import requests  # safe: .venv deps are available
```

**How it works**:
1. System Python runs the script first
2. `Path(__file__).resolve().parent.parent` resolves to the project root (one level above `src/`)
3. If `.venv/bin/python` exists there and is not the current interpreter, `os.execv` replaces the current process — no subprocess, no overhead
4. The script re-runs from the top, this time already inside `.venv`, so the check is skipped

### Script Template

```python
#!/usr/bin/env python3
"""One-line description of what this script does."""

import os
import sys
from pathlib import Path

# Auto-switch to ../.venv
_venv_python = Path(__file__).resolve().parent.parent / ".venv" / "bin" / "python"
if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# Imports (safe: running inside .venv)
import argparse


def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("input", help="Input value")
    args = parser.parse_args()

    print(args.input)


if __name__ == "__main__":
    main()
```

### Make Executable

```bash
chmod +x src/main.py
./src/main.py --help
```

### Multiple Scripts

Each script in `src/` is self-contained. Copy the relauncher block at the top of every script that needs `.venv` deps:

```python
#!/usr/bin/env python3
"""Another script."""

import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent.parent / ".venv" / "bin" / "python"
if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# script-specific code below
from processor import process  # relative import from src/
```

### Why Not Other Approaches

| Approach | Problem |
|----------|---------|
| `#!../.venv/bin/python` | Relative shebags not supported on Linux |
| `#!/usr/bin/env python3` alone | Uses system Python, deps not available |
| Activate venv manually | Requires user to remember every time |
| Bash wrapper script | Extra file, more complexity |
| `#!/absolute/path/.venv/bin/python` | Path changes per machine, not portable |

---

## src/ Layout

### Files Directly in src/

The main files of the project live directly in `src/`:

```
src/
├── main.py        ← Entry point: has shebag + arg parsing (thin)
├── processor.py   ← Core business logic
└── errors.py      ← Custom exception types
```

### When to Add a Sub-Package

Only add a sub-folder when it groups **multiple files** forming a cohesive module:

```
src/
├── main.py
├── processor.py
├── errors.py
└── models/        ← Sub-package: multiple model types together
    ├── __init__.py
    ├── user.py
    └── item.py
```

Do **not** create `utils/`, `helpers/`, `common/` — use domain-specific names.

### `main.py` — Entry Point

```python
#!/usr/bin/env python3
"""Command-line interface for project-name."""

import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent.parent / ".venv" / "bin" / "python"
if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

import argparse

from processor import process
from errors import ProcessingError, ValidationError


def main() -> None:
    """Entry point for the CLI."""
    parser = argparse.ArgumentParser(description="What the tool does")
    parser.add_argument("input", help="Input file path")
    parser.add_argument("--verbose", "-v", action="store_true")
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

### `processor.py` — Business Logic

```python
# src/processor.py
"""Core processing logic."""

from pathlib import Path

from errors import ProcessingError, ValidationError


def process(input_path: str) -> str:
    """Process the given input file and return the result.

    Args:
        input_path: Path to the input file.

    Returns:
        Processed result as a string.

    Raises:
        ValidationError: If input_path is empty or file doesn't exist.
        ProcessingError: If processing fails for any reason.
    """
    if not input_path:
        raise ValidationError("input_path", "cannot be empty")

    path = Path(input_path)
    if not path.exists():
        raise ValidationError("input_path", f"file not found: {input_path}")

    try:
        content = path.read_text(encoding="utf-8")
        # actual processing...
        return content
    except OSError as e:
        raise ProcessingError(f"failed to read file: {e}") from e
```

### `errors.py` — Custom Exceptions

```python
# src/errors.py
"""Custom exception types for project-name."""


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

---

## Tests

### Structure

Tests live in `tests/`, mirroring the `src/` structure:

```
tests/
├── conftest.py               ← Shared fixtures
├── test_main.py
├── test_processor.py
└── models/                   ← Mirror sub-packages if they exist in src/
    └── test_item.py
```

### `conftest.py` — Shared Fixtures

```python
# tests/conftest.py
"""Shared pytest fixtures."""

import pytest
from pathlib import Path


@pytest.fixture
def sample_input(tmp_path: Path) -> Path:
    """Create a temporary input file for testing."""
    f = tmp_path / "input.txt"
    f.write_text("sample content", encoding="utf-8")
    return f
```

### Table-Style Tests with pytest

```python
# tests/test_processor.py
"""Tests for processor."""

import sys
import os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "src"))

import pytest
from pathlib import Path

from processor import process
from errors import ValidationError, ProcessingError


class TestProcess:
    def test_valid_input(self, sample_input: Path) -> None:
        result = process(str(sample_input))
        assert result == "sample content"

    @pytest.mark.parametrize("bad_input,expected_field", [
        ("", "input_path"),
        ("/nonexistent/path.txt", "input_path"),
    ])
    def test_invalid_input(self, bad_input: str, expected_field: str) -> None:
        with pytest.raises(ValidationError) as exc_info:
            process(bad_input)
        assert exc_info.value.field == expected_field
```

Or configure `pythonpath` in `pyproject.toml` to avoid the `sys.path` hack:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["src"]
addopts = "-v --tb=short"
```

### Running Tests

```bash
# All tests
make test

# With coverage
.venv/bin/pytest --cov=src --cov-report=term-missing

# Specific file
.venv/bin/pytest tests/test_processor.py -v

# Specific test
.venv/bin/pytest tests/test_processor.py::TestProcess::test_valid_input -v
```

---

## Error Handling Patterns

### Always Use Custom Exceptions

```python
# ✅ GOOD: Custom hierarchy, easy to catch specifically
try:
    result = process(path)
except ValidationError as e:
    print(f"Bad input: {e}", file=sys.stderr)
    sys.exit(1)
except ProcessingError as e:
    print(f"Processing failed: {e}", file=sys.stderr)
    sys.exit(1)

# ❌ BAD: Catching bare Exception hides bugs
try:
    result = process(path)
except Exception as e:
    print(f"Error: {e}")
```

### Always Chain Exceptions

```python
# ✅ GOOD: preserves original traceback
try:
    data = json.loads(content)
except json.JSONDecodeError as e:
    raise ProcessingError(f"invalid JSON in {path}") from e

# ❌ BAD: loses original traceback
try:
    data = json.loads(content)
except json.JSONDecodeError:
    raise ProcessingError("invalid JSON")
```

### Type Hints and Return Types

```python
# ✅ GOOD: explicit types
def load_config(path: str) -> dict[str, str]:
    ...

# ❌ BAD: no hints
def load_config(path):
    ...
```

---

## .gitignore

```gitignore
# Virtual environment
.venv/

# Build artifacts
dist/
build/
*.egg-info/
src/*.egg-info/

# Python cache
__pycache__/
*.pyc
*.pyo
*.pyd

# Test coverage
.coverage
htmlcov/
.pytest_cache/

# Type checker
.mypy_cache/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db
```

---

## Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Committing .venv

```bash
# ❌ BAD
git add .venv/

# ✅ GOOD: .venv in .gitignore, reproducible via run/setup.sh
```

### ❌ Anti-Pattern 2: Creating Unnecessary Sub-Packages

```
# ❌ BAD: wrapping everything in a package when flat files suffice
src/
└── project_name/
    ├── __init__.py
    └── cli.py

# ✅ GOOD: flat files directly in src/
src/
└── main.py
```

### ❌ Anti-Pattern 3: Logic in main.py

```python
# ❌ BAD: business logic in entry point
def main() -> None:
    args = parse_args()
    data = open(args.input).read()
    result = data.upper()  # logic here!
    print(result)

# ✅ GOOD: entry point only handles I/O and calls modules
def main() -> None:
    args = parse_args()
    result = processor.process(args.input)
    print(result)
```

### ❌ Anti-Pattern 4: Bare except or Ignoring Errors

```python
# ❌ BAD
try:
    result = process(path)
except:
    pass

# ✅ GOOD
try:
    result = process(path)
except ProcessingError as e:
    raise  # or handle explicitly
```

### ❌ Anti-Pattern 5: setup.py Instead of pyproject.toml

```python
# ❌ BAD: legacy approach
# setup.py

# ✅ GOOD: modern standard
# pyproject.toml
```

### ❌ Anti-Pattern 6: Generic Module Names

```python
# ❌ BAD
src/utils.py
src/helpers.py
src/common.py

# ✅ GOOD: domain-specific names
src/processor.py
src/validator.py
src/reader.py
```

### ❌ Anti-Pattern 7: Relative shebag

```python
# ❌ BAD: not supported by Linux kernel
#!/.venv/bin/python
#!../.venv/bin/python

# ✅ GOOD: self-relaunching pattern with ../.venv
#!/usr/bin/env python3
# + the os.execv relauncher block
```

---

## Key Principles

### 1. Virtual Environment
- Always `.venv` at project root
- Scripts in `src/` use `parent.parent / ".venv"` to find it
- Reproducible via `run/setup.sh`

### 2. Dependencies
- Runtime deps in `[project.dependencies]`
- Dev deps in `[project.optional-dependencies] dev`
- Always pin minimum versions (`>=`)

### 3. File Organization
- Main files directly in `src/`
- Sub-packages only when multiple files form a true module
- Tests in `tests/`, mirroring `src/` structure

### 4. Testing
- Use `pythonpath = ["src"]` in `pyproject.toml` to avoid `sys.path` manipulation
- Use `conftest.py` for shared fixtures
- Parametrize to avoid test duplication
- Aim for 80%+ coverage

### 5. Code Quality
- `ruff` for linting and formatting
- `mypy` with `strict = true` for type checking
- Run both before committing

---

## Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add new entry point | `src/new-script.py` | Thin: parse args, call modules |
| Add business logic | `src/processor.py` or domain-named file | Keep thin layers |
| Add custom exception | `src/errors.py` | Extend `ProjectError` |
| Add multiple related types | `src/models/` | Only if multiple files are needed |
| Add test | `tests/` mirroring `src/` | One test file per source file |
| Add test fixture | `tests/conftest.py` | Shared across test files |
| Add runtime dependency | `pyproject.toml [project.dependencies]` | Then `make setup` |
| Add dev dependency | `pyproject.toml [project.optional-dependencies] dev` | Then `make setup` |

---

## Development Workflow

```bash
# First time setup
make setup

# Daily loop
make fmt          # Format code
make lint         # Check for issues
make typecheck    # Type check
make test         # Run tests

# Run a script directly
./src/main.py --help

# Before committing
make lint
make test

# Install script locally
make install
project-name --help
```

---

## Related Skills

- **go-project-structure** — Same philosophy, adapted for Go
- **git-workflow-go** — Commit conventions apply equally to Python projects
- **readme-bilingual-sync** — Keep README.md and README-PT.md synchronized
