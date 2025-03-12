---
name: python-project-structure
description: Guide to organizing Python projects with pyproject.toml, .venv, src/ layout, Makefile hierarchy, and run/ scripts. Covers anti-patterns, testing patterns, error handling, and works for multiple independent Python projects.
mode: agent
category: python
shared: true
---

# Python Project Structure

Comprehensive guide to organizing Python projects following a consistent pattern for maintainability and scalability across multiple independent projects.

## Overview

This skill explains a pragmatic Python project structure where all source code lives in `src/<package_name>/`, the virtual environment stays at the project root as `.venv`, and `pyproject.toml` defines the project. Uses a hierarchical Makefile approach: root Makefile for orchestration, `run/` scripts for the actual work.

**Key principles**:
- `.venv` is always at the project root — never inside `src/`
- `pyproject.toml` is the single source of truth for metadata and dependencies
- `src/` layout prevents accidental imports without installation
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
│   └── install.sh                ← Install CLI tool to ~/.local/bin
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
├── src/                          ← All Python source code
│   └── project_name/             ← Main package (use underscores)
│       ├── __init__.py           ← Package init (keep minimal)
│       ├── cli.py                ← CLI entry point
│       ├── core/                 ← Core business logic
│       │   ├── __init__.py
│       │   ├── processor.py      ← Main processing logic
│       │   └── errors.py         ← Custom exception types
│       └── utils/                ← Only if domain-specific name doesn't fit
│           ├── __init__.py
│           └── helpers.py
│
└── tests/                        ← All tests (outside src/)
    ├── __init__.py
    ├── conftest.py               ← Pytest fixtures and configuration
    ├── test_processor.py         ← Tests mirror src/ structure
    └── test_cli.py
```

---

## Core Files

### `pyproject.toml` — Project Definition

**Location**: Project root (never inside `src/`).

**Purpose**: Single file for metadata, dependencies, build system, and tool configuration.

**Example**:
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

[project.scripts]
project-name = "project_name.cli:main"

[tool.setuptools.packages.find]
where = ["src"]

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

**Install project in editable mode** (so imports from `src/` work):
```bash
pip install -e ".[dev]"
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
	@echo "  install    Install CLI tool to ~/.local/bin"
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

BINARY_NAME="project-name"
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/.venv/bin/$BINARY_NAME" "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

---

## src/ Layout

### Why src/ Layout?

Without `src/` layout, Python can import your package directly from the project root, even without installing it. This masks missing dependencies and import errors.

With `src/` layout:
- Package must be installed (`pip install -e .`) to be importable
- Catches missing `__init__.py` files early
- Mirrors how the package behaves when installed by end users

### Package Naming

Use **underscores** for Python package names (directory inside `src/`):

```
project-name/     ← project directory (hyphens ok)
└── src/
    └── project_name/   ← Python package (underscores required)
```

### `__init__.py`

Keep it minimal — only expose the public API:

```python
# src/project_name/__init__.py
"""Project Name — short description."""

__version__ = "0.1.0"
```

### CLI Entry Point

```python
# src/project_name/cli.py
"""Command-line interface for project-name."""

import argparse
import sys

from project_name.core.processor import process


def main() -> None:
    """Entry point for the CLI."""
    parser = argparse.ArgumentParser(description="What the tool does")
    parser.add_argument("input", help="Input file path")
    parser.add_argument("--verbose", "-v", action="store_true")
    args = parser.parse_args()

    try:
        result = process(args.input)
        print(result)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)
```

---

## Core Package Structure

### `core/processor.py` — Business Logic

```python
# src/project_name/core/processor.py
"""Core processing logic."""

from pathlib import Path

from project_name.core.errors import ProcessingError, ValidationError


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

### `core/errors.py` — Custom Exceptions

```python
# src/project_name/core/errors.py
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

## Shebangs and .venv

Python scripts meant to be run directly (e.g. `./script.py`) must point to the right interpreter. On Linux, shebangs **must be absolute paths** — relative paths like `#!.venv/bin/python` are not supported by the kernel.

The solution is the **self-relaunching pattern**: the script starts with the system Python, detects the `.venv` in the project root, and re-executes itself using the `.venv` interpreter if not already running inside it.

### The Self-Relaunching Pattern

```python
#!/usr/bin/env python3
"""Script description."""

# Auto-switch to .venv if available
import os
import sys
from pathlib import Path

_root = Path(__file__).resolve().parent
_venv_python = _root / ".venv" / "bin" / "python"

if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# --- script starts here, guaranteed to be running inside .venv ---

import requests  # safe: .venv deps are available
```

**How it works**:
1. System Python runs the script first
2. `__file__` resolves to the script's location, which is inside the project root
3. If `.venv/bin/python` exists and is not the current interpreter, `os.execv` replaces the current process with the `.venv` Python — no subprocess, no overhead
4. The script re-runs from the top, this time already inside `.venv`, so the check is skipped

### Where to Place Scripts

Scripts meant to be run directly go at the project root, not inside `src/`:

```
project-name/
├── .venv/
├── pyproject.toml
├── my-script.py       ← executable script, run with ./my-script.py
├── src/
│   └── project_name/  ← importable package
└── tests/
```

Make executable:
```bash
chmod +x my-script.py
./my-script.py
```

### Multiple Scripts

Each script is self-contained. Copy the relauncher block at the top of every script:

```python
#!/usr/bin/env python3
"""Another script."""

import os
import sys
from pathlib import Path

_root = Path(__file__).resolve().parent
_venv_python = _root / ".venv" / "bin" / "python"

if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# script-specific code below
from project_name.core.processor import process
```

### Why Not Other Approaches

| Approach | Problem |
|----------|---------|
| `#!.venv/bin/python` | Relative shebangs not supported on Linux |
| `#!/usr/bin/env python3` alone | Uses system Python, deps not available |
| Activate venv manually | Requires user to remember every time |
| Bash wrapper script | Extra file, more complexity |
| `#!/absolute/path/.venv/bin/python` | Path changes per machine, not portable |

### Script Template

```python
#!/usr/bin/env python3
"""One-line description of what this script does."""

import os
import sys
from pathlib import Path

# Auto-switch to .venv
_root = Path(__file__).resolve().parent
_venv_python = _root / ".venv" / "bin" / "python"
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

---

## Tests

### Structure

Tests live in `tests/`, mirroring the `src/` structure:

```
tests/
├── __init__.py
├── conftest.py               ← Shared fixtures
├── test_cli.py               ← Tests for cli.py
└── core/
    ├── __init__.py
    └── test_processor.py     ← Tests for core/processor.py
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
# tests/core/test_processor.py
"""Tests for core.processor."""

import pytest
from pathlib import Path

from project_name.core.processor import process
from project_name.core.errors import ValidationError, ProcessingError


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

### Running Tests

```bash
# All tests
make test

# With coverage
.venv/bin/pytest --cov=src/project_name --cov-report=term-missing

# Specific file
.venv/bin/pytest tests/core/test_processor.py -v

# Specific test
.venv/bin/pytest tests/core/test_processor.py::TestProcess::test_valid_input -v
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

### ❌ Anti-Pattern 2: No src/ Layout

```
# ❌ BAD: package at root, importable without installing
project-name/
└── project_name/
    └── __init__.py

# ✅ GOOD: src/ layout forces proper installation
project-name/
└── src/
    └── project_name/
        └── __init__.py
```

### ❌ Anti-Pattern 3: Logic in cli.py

```python
# ❌ BAD: business logic in CLI
def main() -> None:
    args = parse_args()
    data = open(args.input).read()
    result = data.upper()  # logic here!
    print(result)

# ✅ GOOD: CLI only handles I/O and calls core
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
src/project_name/utils.py
src/project_name/helpers.py
src/project_name/common.py

# ✅ GOOD: domain-specific names
src/project_name/core/processor.py
src/project_name/core/validator.py
src/project_name/io/reader.py
```

---

## Key Principles

### 1. Virtual Environment
- Always `.venv` at project root
- Always activate before running anything: `source .venv/bin/activate`
- Reproducible via `run/setup.sh`

### 2. Dependencies
- Runtime deps in `[project.dependencies]`
- Dev deps in `[project.optional-dependencies] dev`
- Always pin minimum versions (`>=`)
- Lock with `pip freeze > requirements.lock` if needed

### 3. Package Naming
- Directory: `project-name/` (hyphens ok)
- Python package: `project_name/` (underscores required)
- Entry point: defined in `[project.scripts]`

### 4. Testing
- Tests in `tests/`, not inside `src/`
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
| Add new CLI command | `src/project_name/cli.py` | Extend `main()` with subcommands |
| Add business logic | `src/project_name/core/` | Keep thin layers |
| Add custom exception | `src/project_name/core/errors.py` | Extend `ProjectError` |
| Add test | `tests/` mirroring `src/` | One test file per source file |
| Add test fixture | `tests/conftest.py` | Shared across test files |
| Add runtime dependency | `pyproject.toml [project.dependencies]` | Then `make setup` |
| Add dev dependency | `pyproject.toml [project.optional-dependencies] dev` | Then `make setup` |

---

## Development Workflow

```bash
# First time setup
make setup
source .venv/bin/activate

# Daily loop
make fmt          # Format code
make lint         # Check for issues
make typecheck    # Type check
make test         # Run tests

# Before committing
make lint
make test

# Install CLI locally
make install
project-name --help
```

---

## Related Skills

- **go-project-structure** — Same philosophy, adapted for Go
- **git-workflow-go** — Commit conventions apply equally to Python projects
- **readme-bilingual-sync** — Keep README.md and README-PT.md synchronized
