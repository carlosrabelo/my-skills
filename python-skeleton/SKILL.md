---
name: python-skeleton
description: Standard Python project structure (pyproject.toml at root, project_name/ package for all source, python -m execution). Creates from scratch or reorganizes existing projects.
mode: agent
category: python
shared: true
---

# Python Skeleton

Unified skill for organizing Python projects following a consistent pattern for maintainability and scalability. Handles both new projects and reorganization of existing ones.

## Core Rule

> **All `.py` source files live inside `project_name/`. The project root contains only infrastructure.**

```
project-name/          ← root: infrastructure only
├── pyproject.toml     ← project metadata and tool config
├── Makefile           ← orchestration
├── make/              ← build/test/lint scripts
├── bin/               ← scripts/wrappers (optional)
├── docs/              ← documentation (optional)
├── tests/             ← test files
├── README.md
└── project_name/      ← ALL source code here
    ├── __init__.py
    ├── cli.py         ← entry point (thin)
    └── *.py
```

This rule applies to **both new projects and migrations**. No loose `.py` files at the project root — ever.

## Context Detection

Before starting, determine the context:

1. **Check for `pyproject.toml`** in the current directory (or the target directory):
   ```bash
   find . -name "pyproject.toml" -not -path "./.venv/*" -maxdepth 3
   ```

2. **If `pyproject.toml` exists** → this is an existing project that needs reorganization. Follow **## Migrating an Existing Python Project** below.

3. **If `pyproject.toml` does not exist** (or the directory is empty/new) → this is a new project being created from scratch. Follow **## Creating from Scratch** below.

4. **Always apply the canonical layout** defined in **## Canonical Layout** below — it is the target structure for both flows.

5. **Apply patterns** from **## Patterns** when the task involves testing, error handling, anti-patterns, or running the application.

---

## Migrating an Existing Python Project

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the project looks "mostly correct".**

### Mandatory Checklist

#### Before Starting
- [ ] `git status` is clean
- [ ] All tests pass
- [ ] Understand what the project does
- [ ] Identify what exists and what's missing

#### Diagnose the Project

```bash
# Where are the Python files?
find . -name "*.py" -not -path "./.venv/*" | head -20

# How big are the modules?
wc -l *.py

# What's inside utils.py (if it exists)?
grep "^def \|^class " utils.py

# Are there sub-packages?
find . -name "__init__.py" -not -path "./.venv/*"

# Are there imports from src/?
grep -r "from src" *.py tests/
```

#### During Reorganization
- [ ] Move files to correct locations
- [ ] Update all imports (moved code + consuming code)
- [ ] Remove `sys.path` hacks if present
- [ ] Tests still pass after each move

#### After Reorganization
- [ ] `make test` passes
- [ ] `make typecheck` passes
- [ ] `make lint` passes
- [ ] Entry point ≤ 50 lines
- [ ] All source files inside `project_name/`
- [ ] No loose `.py` files at project root
- [ ] No `utils.py` or `helpers.py`
- [ ] No sub-packages wrapping single files
- [ ] Commit with clear message

#### Rules
- **Never mix reorganization with logic changes** — reorganize first, then modify behavior in a separate commit
- **Move one file at a time** — move, update imports, verify tests, repeat
- **All source files belong in `project_name/`** — no loose `.py` files at project root
- **Only move existing files** — do not create new `.py` modules inside `project_name/` that did not exist before; the goal is to separate code from project infrastructure, not to introduce new structure
- **No generic module names** — `utils.py`, `helpers.py`, `common.py` must be renamed to domain-specific names
- **No sub-packages for single files** — flatten inside `project_name/` instead
- **Entry point must be thin** — ≤50 lines, CLI parsing + delegation only
- **Use relative imports** — inside the package use `from .module import thing`

---

### Migration Scenarios

#### Scenario 1: Migrate from src/ Layout

**Symptom**: Python files inside `src/`, `pyproject.toml` has `pythonpath = ["src"]`.

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

1. **Create the package directory and move source files**:
```bash
mkdir project_name
touch project_name/__init__.py
mv src/*.py project_name/
# Move sub-packages if any
# mv src/models/ project_name/
rmdir src/
```

2. **Convert absolute imports to relative** inside all moved files:
```python
# Old: from errors import ProcessingError
# New:
from .errors import ProcessingError
```

3. **Update `pyproject.toml`**:
```toml
[project.scripts]
project-name = "project_name.cli:main"

[tool.setuptools.packages.find]
where = ["."]
include = ["project_name*"]

[tool.ruff]
src = ["."]

[tool.pytest.ini_options]
pythonpath = ["."]
```

4. **Update `Makefile`**:
```makefile
PY_SOURCES := project_name/

fmt:
	.venv/bin/ruff format $(PY_SOURCES) tests/

typecheck:
	.venv/bin/mypy $(PY_SOURCES)
```

5. **Update `make/lint.sh`**:
```bash
# Old: "$ROOT_DIR/.venv/bin/ruff" check src/ tests/
# New:
"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR/project_name/" tests/
```

6. **Remove `sys.path` hacks from tests** and update imports:
```python
# Delete these lines:
# import sys
# import os
# sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "src"))

# Update imports:
# Old: from fetcher import main
# New: from project_name.cli import main
```

7. **Verify**:
```bash
make test
make lint
make typecheck
```

---

#### Scenario 2: Monolithic main.py

**Symptom**: `main.py` is 200+ lines with business logic mixed into the entry point.

**Steps**:

1. **Identify concerns** in main.py: CLI/argument parsing, business/processing logic, error types, output formatting

2. **Create domain-specific modules**:
```bash
touch processor.py errors.py
touch tests/test_processor.py
```

3. **Define errors in `errors.py`**:
```python
# errors.py
"""Custom exception types."""


class ProjectError(Exception):
    """Base exception for all project errors."""


class ValidationError(ProjectError):
    def __init__(self, field: str, message: str) -> None:
        self.field = field
        self.message = message
        super().__init__(f"validation failed on {field}: {message}")


class ProcessingError(ProjectError):
    """Raised when processing fails."""
```

4. **Move business logic** to `processor.py`, update imports

5. **Slim down main.py** to ≤50 lines: argument parsing + delegation only

6. **Verify**:
```bash
make test
make lint
```

**After**:
```
project-name/
├── main.py       ← ~30 lines (CLI only)
├── processor.py  ← business logic
└── errors.py     ← custom exceptions
```

---

#### Scenario 3: Oversized Module

**Symptom**: A module is 400+ lines mixing multiple concerns.

**Steps**:

1. **Identify concerns** in the file (loading, processing, formatting, etc.)

2. **Create one file per concern**:
```bash
touch loader.py formatter.py
touch tests/test_loader.py tests/test_formatter.py
```

3. **Extract each concern** to its own file with a focused module docstring

4. **Keep the original file as thin public interface** (or delete if not needed):
```python
# processor.py — delegates to focused modules
from loader import load_input
from formatter import format_result
```

5. **Verify**:
```bash
make test
wc -l *.py
```

---

#### Scenario 4: Generic Module Names

**Symptom**: File named `utils.py`, `helpers.py`, or `common.py` with mixed concerns.

**Steps**:

1. **Analyze what's inside**:
```bash
grep "^def \|^class " utils.py
```

2. **Group functions by domain** — text ops, time ops, path ops, etc.

3. **Create domain-specific modules** and move functions

4. **Update all imports** across the codebase:
```bash
grep -r "from utils import" *.py tests/
grep -r "import utils" *.py tests/
```

5. **Delete old generic module**:
```bash
rm utils.py
```

6. **Verify**:
```bash
make test
make typecheck
```

---

#### Scenario 5: Duplicated Code Across Modules

**Symptom**: Same function or logic copy-pasted in multiple files.

**Steps**:

1. **Compare the duplicated code**:
```bash
grep -A 10 "def validate_path" convert.py
grep -A 10 "def validate_path" export.py
```

2. **Create a shared module** inside `project_name/` (e.g., `project_name/validation.py`)

3. **Move the common implementation** to the shared module

4. **Update all importing files** to use the shared module

5. **Delete duplicate code** from original files

6. **Verify**:
```bash
make test
```

---

#### Scenario 6: Flatten Unnecessary Sub-Package

**Symptom**: A sub-package directory contains only one file.

**Before**:
```
project-name/
└── models/
    ├── __init__.py
    └── item.py       ← only one file
```

**Steps**:

1. **Move file to the parent level inside `project_name/`**:
```bash
mv project_name/models/item.py project_name/item.py
rm project_name/models/__init__.py
rmdir project_name/models/
```

2. **Update all imports**:
```bash
grep -r "from .models" project_name/ tests/
# Old: from .models.item import Item
# New: from .item import Item
```

3. **Move tests** if nested:
```bash
mv tests/models/test_item.py tests/test_item.py
rmdir tests/models/
```

4. **Verify**:
```bash
make test
```

---

#### Scenario 7: Introduce Sub-Package When Justified

**Symptom**: 5+ files of the same domain inside `project_name/` are hard to distinguish from unrelated logic files.

**Steps**:

1. **Create the sub-package inside `project_name/`**:
```bash
mkdir -p project_name/models
touch project_name/models/__init__.py
```

2. **Move the files** and update imports:
```bash
mv project_name/user.py project_name/item.py project_name/order.py project_name/models/
# Old: from .user import User
# New: from .models.user import User
```

3. **Move tests**:
```bash
mkdir -p tests/models
touch tests/models/__init__.py
mv tests/test_user.py tests/test_item.py tests/models/
```

4. **Verify**:
```bash
make test
```

---

#### Scenario 8: Add Proper Test Fixtures

**Symptom**: Tests use hardcoded paths, `/tmp/` files, or duplicated setup logic.

**Steps**:

1. **Create `tests/conftest.py`** with shared fixtures:
```python
# tests/conftest.py
"""Shared pytest fixtures."""

import pytest
from pathlib import Path


@pytest.fixture
def sample_text_file(tmp_path: Path) -> Path:
    f = tmp_path / "input.txt"
    f.write_text("line1\nline2\nline3", encoding="utf-8")
    return f
```

2. **Create `tests/fixtures/`** for static test data:
```bash
mkdir -p tests/fixtures
echo '{"name": "test", "value": 42}' > tests/fixtures/valid.json
```

3. **Add a fixture to expose the fixtures directory**:
```python
FIXTURES_DIR = Path(__file__).parent / "fixtures"


@pytest.fixture
def fixtures_dir() -> Path:
    return FIXTURES_DIR
```

4. **Update tests** to use fixtures instead of hardcoded values

---

#### Scenario 9: Migrate from Flat Root to project_name/ Package

**Symptom**: Python source files scattered at the project root alongside `Makefile`, `README.md`, `pyproject.toml`.

**Before**:
```
project-name/
├── fetcher.py
├── processor.py
├── errors.py
├── tests/
└── pyproject.toml
```

**Steps**:

1. **Create the package directory**:
```bash
mkdir project_name
touch project_name/__init__.py
```

2. **Move source files into the package**:
```bash
mv fetcher.py processor.py errors.py project_name/
# Move any sub-packages too:
# mv models/ project_name/
```

3. **Convert absolute imports to relative** inside all moved files:
```python
# Old (in processor.py, now project_name/processor.py):
from errors import ProcessingError

# New:
from .errors import ProcessingError
```

4. **Update entry point** (rename to `cli.py` if it was `main.py` or `fetcher.py`):
```python
# Old (fetcher.py):
from processor import process

# New (project_name/cli.py):
from .processor import process
```

5. **Update `pyproject.toml`**:
```toml
[project.scripts]
project-name = "project_name.cli:main"   # was: fetcher:main

[tool.setuptools.packages.find]
where = ["."]
include = ["project_name*"]

[tool.pytest.ini_options]
pythonpath = ["."]   # unchanged
```

6. **Update `Makefile`**:
```makefile
# Old:
PY_FILES := $(wildcard *.py)

# New:
PY_SOURCES := project_name/
```
And update `fmt`, `typecheck`, `run` targets to use `$(PY_SOURCES)` and `project_name.cli`.

7. **Update `make/lint.sh`**:
```bash
# Old: "$ROOT_DIR"/*.py
# New:
"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR/project_name/" tests/
```

8. **Update test imports**:
```python
# Old: from processor import process
# New: from project_name.processor import process
```

9. **Verify**:
```bash
make test
make lint
make typecheck
```

---

#### Scenario 10: Migrate requirements.txt to pyproject.toml

**Symptom**: Project uses `requirements.txt` for dependencies instead of `pyproject.toml`.

**Steps**:

1. **Create `pyproject.toml`** at the project root with dependencies from `requirements.txt` (see **## Canonical Layout** below for the full template)

2. **Update `make/setup.sh`** to install from `pyproject.toml`:
```bash
# Old:
"$ROOT_DIR/.venv/bin/pip" install -r "$ROOT_DIR/requirements.txt"

# New:
"$ROOT_DIR/.venv/bin/pip" install -e "$ROOT_DIR[dev]" --quiet
```

3. **Handle `requirements.txt`**:
```bash
# Option A: keep as lockfile
pip freeze > requirements.txt

# Option B: remove it
rm requirements.txt
```

4. **Verify**:
```bash
rm -rf .venv
make setup    # rebuilds .venv from pyproject.toml
make test
```

---

## Creating from Scratch

### Steps

1. **Determine `project_name`** — replace hyphens with underscores: `my-project` → `my_project`

2. **Create directories and package**:
```bash
mkdir -p project_name make tests
touch project_name/__init__.py
```

3. **Create entry point `project_name/cli.py`**:
```python
"""Command-line interface for project-name."""
import argparse
import sys


def main() -> None:
    parser = argparse.ArgumentParser(description="What this tool does")
    parser.add_argument("input", help="Input value")
    args = parser.parse_args()

    print(args.input)


if __name__ == "__main__":
    main()
```

4. **Create `pyproject.toml`** at root:
```toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "project-name"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
project-name = "project_name.cli:main"

[tool.setuptools.packages.find]
where = ["."]
include = ["project_name*"]

[tool.pytest.ini_options]
pythonpath = ["."]

[tool.ruff]
src = ["."]
```

5. **Create virtualenv and install**:
```bash
python3 -m venv .venv
.venv/bin/pip install -e ".[dev]"
```

6. **Create `make/test.sh`**:
```bash
cat > make/test.sh <<'EOF'
#!/bin/bash
set -euo pipefail
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"
.venv/bin/pytest -v
EOF
chmod +x make/test.sh
```

7. **Create root Makefile** (see ## Canonical Layout for full template)

8. **Verify**:
```bash
.venv/bin/python -m project_name.cli --help
```

---

### Project Layout

All source files live inside a directory named after the project (underscores, not hyphens). The entry point is named after what it does:

```
project-name/
└── project_name/        ← all source code here
    ├── __init__.py
    ├── cli.py            ← Entry point: arg parsing (thin). Name matches the domain.
    ├── processor.py      ← Core business logic
    └── errors.py         ← Custom exception types
```

Use `cli.py` as the entry point when no better domain name exists. Good names: `cli.py`, `fetcher.py`, `converter.py`, `scraper.py`, `importer.py`.

#### When to Add a Sub-Package

Only add a sub-folder inside `project_name/` when it groups **multiple files** forming a cohesive module:

```
project-name/
└── project_name/
    ├── __init__.py
    ├── cli.py
    ├── processor.py
    ├── errors.py
    └── models/        ← Sub-package: multiple model types together
        ├── __init__.py
        ├── user.py
        └── item.py
```

Do **not** create `utils/`, `helpers/`, `common/` — use domain-specific names.

---

### Entry Point Template

```python
# project_name/cli.py
"""Command-line interface for project-name."""

import argparse
import sys

from .processor import process
from .errors import ProcessingError, ValidationError


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

```bash
.venv/bin/python -m project_name.cli --help
```

---

### `processor.py` — Business Logic

```python
# project_name/processor.py
"""Core processing logic."""

from pathlib import Path

from .errors import ProcessingError, ValidationError


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

---

### `errors.py` — Custom Exceptions

```python
# project_name/errors.py
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

### Key Principles

#### 1. Virtual Environment
- Always `.venv` at project root
- Reproducible via `make/setup.sh`

#### 2. Dependencies
- Runtime deps in `[project.dependencies]`
- Dev deps in `[project.optional-dependencies] dev`
- Always pin minimum versions (`>=`)

#### 3. File Organization
- Source files live in `project_name/` — a directory named after the project (using underscores), containing `__init__.py`
- Use relative imports inside the package (`from .processor import process`)
- Sub-packages inside `project_name/` only when multiple files form a cohesive module (e.g. `project_name/models/`)
- Tests in `tests/`, mirroring the `project_name/` structure

#### 4. Testing
- Use `pythonpath = ["."]` in `pyproject.toml` — no `sys.path` manipulation needed
- Use `conftest.py` for shared fixtures
- Parametrize to avoid test duplication
- Aim for 80%+ coverage

#### 5. Code Quality
- `ruff` for linting and formatting
- `mypy` with `strict = true` for type checking
- Run both before committing

---

### Directory Decision Matrix

| Need | Location | Notes |
|------|----------|-------|
| Add new entry point | `project_name/cli.py` or domain-named file | Thin: parse args, call modules |
| Add business logic | `project_name/processor.py` or domain-named file | Keep thin layers |
| Add custom exception | `project_name/errors.py` | Extend `ProjectError` |
| Add multiple related types | `project_name/models/` | Only if multiple files are needed |
| Add test | `tests/` mirroring `project_name/` structure | One test file per source file |
| Add test fixture | `tests/conftest.py` | Shared across test files |
| Add runtime dependency | `pyproject.toml [project.dependencies]` | Then `make setup` |
| Add dev dependency | `pyproject.toml [project.optional-dependencies] dev` | Then `make setup` |

---

### Development Workflow

```bash
# First time setup
make setup

# Daily loop
make fmt          # Format code
make lint         # Check for issues
make typecheck    # Type check
make test         # Run tests

# Run the application (use actual package.module)
.venv/bin/python -m project_name.cli --help
make run

# Before committing
make lint
make test

# Install locally (requires [project.scripts] in pyproject.toml)
make install
project-name --help
```

---

## Canonical Layout

All source files live inside `project_name/`. The virtual environment stays as `.venv` at the project root. `pyproject.toml` defines the project.

**Key principles**:
- `.venv` is always at the project root — a sibling of `project_name/`
- `pyproject.toml` is the single source of truth for metadata and dependencies
- All source files live inside `project_name/` — no loose `.py` files at the project root
- The main entry point can be named anything (`main.py`, `fetcher.py`, `cli.py`, etc.) — use a domain-specific name, not necessarily `main.py`
- Applications are run with `python -m <package>.<module>` using `.venv/bin/python`
- Python version: **3.12**

### Standard Project Layout

```
project-name/
├── Makefile                      ← Root Makefile (orchestrates everything)
│
├── .venv/                        ← Virtual environment (NEVER commit)
│
├── pyproject.toml                ← Project metadata, dependencies, tool config
│
├── make/                          ← Automation scripts
│   ├── setup.sh                  ← Create .venv and install dependencies
│   ├── test.sh                   ← Run tests
│   ├── lint.sh                   ← Run linter
│   └── install.sh                ← Install script(s) to ~/.local/bin
│
├── LICENSE                       ← Project license
├── README.md                     ← English documentation
├── README-PT.md                  ← Portuguese documentation
│
├── project_name/                 ← ALL source code (named after the project, underscores)
│   ├── __init__.py
│   ├── cli.py                    ← Entry point (domain-named, run via python -m)
│   ├── processor.py              ← Core logic module
│   ├── errors.py                 ← Custom exception types
│   └── models/                   ← Sub-package ONLY if there are multiple related modules
│       ├── __init__.py
│       └── item.py
│
└── tests/                        ← All tests
    ├── conftest.py               ← Pytest fixtures and configuration
    ├── test_cli.py
    ├── test_processor.py
    └── models/                   ← Mirror sub-packages if they exist
        └── test_item.py
```

> The entry point is named after what it does — `cli.py`, `fetcher.py`, `converter.py`, `scraper.py`, etc. The directory `project_name/` always matches the project folder name (replacing hyphens with underscores).

#### When to Create a Sub-Package

Only create a sub-folder inside `project_name/` when:
- It groups **multiple files** that form a cohesive module (e.g., `models/`, `api/`)
- It needs an `__init__.py` to be importable as a package
- A single file would become too large or have mixed concerns

**Never** create `utils/`, `helpers/`, or `common/` folders — give the module a domain-specific name.

---

### `pyproject.toml` — Project Definition

**Location**: Project root. Single file for metadata, dependencies, build system, and tool configuration.

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

[project.scripts]
project-name = "project_name.cli:main"   # ← use actual package.module:function

[tool.setuptools.packages.find]
where = ["."]
include = ["project_name*"]

[tool.pytest.ini_options]
testpaths = ["tests"]
pythonpath = ["."]
addopts = "-v --tb=short"

[tool.ruff]
src = ["."]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

[tool.mypy]
python_version = "3.12"
strict = true
```

> Always add `[tool.setuptools.packages.find]` — tells setuptools to pick up the `project_name/` package when installing.

---

### `.venv/` — Virtual Environment

**Location**: Always at project root. Never commit.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
# or, for script-only projects:
pip install ".[dev]"
```

---

### Makefile

Single entry point for all operations. Delegates to `make/` scripts.

```makefile
MAKEFLAGS += --no-print-directory

PY_SOURCES := project_name/

.PHONY: setup test lint fmt typecheck clean install run help

setup:
	./make/setup.sh

test:
	./make/test.sh

lint:
	./make/lint.sh

fmt:
	.venv/bin/ruff format $(PY_SOURCES) tests/

typecheck:
	.venv/bin/mypy $(PY_SOURCES)

clean:
	rm -rf dist/ build/ *.egg-info
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete

run:
	.venv/bin/python -m project_name.cli   # ← use actual package.module; append args via: make run ARGS="--help"

install: setup
	./make/install.sh

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
	@echo "  run        Run the application"
	@echo "  install    Install to ~/.local/bin"
```

---

### make/ — Shell Scripts

Scripts do the actual work directly. Always start with `set -euo pipefail`. Activate `.venv` before running Python commands.

**`make/setup.sh`**:
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

**`make/test.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

"$ROOT_DIR/.venv/bin/pytest" "$ROOT_DIR/tests/" "$@"
```

**`make/lint.sh`**:
```bash
#!/bin/bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR/project_name/" tests/
```

**`make/install.sh`**:
```bash
#!/bin/bash
set -euo pipefail

BINARY_NAME="project-name"   # ← must match [project.scripts] key in pyproject.toml
INSTALL_DIR="${HOME}/.local/bin"
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"

cp "$ROOT_DIR/.venv/bin/$BINARY_NAME" "$INSTALL_DIR/$BINARY_NAME"
chmod +x "$INSTALL_DIR/$BINARY_NAME"
echo "Installed: $INSTALL_DIR/$BINARY_NAME"
```

---

### .gitignore

```gitignore
# Virtual environment
.venv/

# Build artifacts
dist/
build/
*.egg-info/

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

## Patterns

### Running the Application

Applications are run with `python -m <package>.<module>` using the project's `.venv`:

```bash
# Standard way to run
.venv/bin/python -m project_name.cli
.venv/bin/python -m project_name.cli --help

# Or via Makefile
make run

# After activating the venv
source .venv/bin/activate
python -m project_name.cli --help
```

The entry point module must have an `if __name__ == "__main__"` block:

```python
# project_name/cli.py
"""Command-line interface for project-name."""

import argparse
import sys

from .processor import process
from .errors import ProcessingError, ValidationError


def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("input", help="Input value")
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

#### Installing as a Command

Add `[project.scripts]` to `pyproject.toml` to make the tool available as a named command after `pip install`:

```toml
[project.scripts]
project-name = "project_name.cli:main"   # ← "package.module:function"
```

After `make setup` (which runs `pip install -e .`), the command is available at `.venv/bin/project-name`. The `make/install.sh` script copies it to `~/.local/bin/`.

---

### Testing Patterns

#### Structure

Tests live in `tests/`, with one test file per source file:

```
tests/
├── conftest.py               ← Shared fixtures
├── test_fetcher.py
├── test_processor.py
└── models/                   ← Mirror sub-packages if they exist
    └── test_item.py
```

#### `conftest.py` — Shared Fixtures

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

#### Table-Style Tests with pytest

With `pythonpath = ["."]` in `pyproject.toml`, package imports work directly — no `sys.path` hack needed:

```python
# tests/test_processor.py
"""Tests for processor."""

import pytest
from pathlib import Path

from project_name.processor import process
from project_name.errors import ValidationError, ProcessingError


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

#### Running Tests

```bash
make test

# With coverage
.venv/bin/pytest --cov=. --cov-report=term-missing

# Specific file
.venv/bin/pytest tests/test_processor.py -v

# Specific test
.venv/bin/pytest tests/test_processor.py::TestProcess::test_valid_input -v
```

---

### Error Handling Patterns

#### Always Use Custom Exceptions

```python
# GOOD: Custom hierarchy, easy to catch specifically
try:
    result = process(path)
except ValidationError as e:
    print(f"Bad input: {e}", file=sys.stderr)
    sys.exit(1)
except ProcessingError as e:
    print(f"Processing failed: {e}", file=sys.stderr)
    sys.exit(1)

# BAD: Catching bare Exception hides bugs
try:
    result = process(path)
except Exception as e:
    print(f"Error: {e}")
```

#### Always Chain Exceptions

```python
# GOOD: preserves original traceback
try:
    data = json.loads(content)
except json.JSONDecodeError as e:
    raise ProcessingError(f"invalid JSON in {path}") from e

# BAD: loses original traceback
try:
    data = json.loads(content)
except json.JSONDecodeError:
    raise ProcessingError("invalid JSON")
```

#### Type Hints and Return Types

```python
# GOOD: explicit types
def load_config(path: str) -> dict[str, str]:
    ...

# BAD: no hints
def load_config(path):
    ...
```

---

### Anti-Patterns: What NOT to Do

#### Anti-Pattern 1: Committing .venv

```bash
# BAD
git add .venv/

# GOOD: .venv in .gitignore, reproducible via make/setup.sh
```

#### Anti-Pattern 2: Source Files at Project Root

Leaving `.py` source files loose at the project root mixes code with Makefile, README, pyproject.toml — no clear boundary between source and project infrastructure.

```
# BAD: source files mixed with project files at root
project-name/
├── fetcher.py        ← code mixed with...
├── processor.py      ← ...Makefile, README, pyproject.toml
├── errors.py
├── Makefile
└── pyproject.toml

# BAD: src/ wrapper (generic name, not tied to the project)
project-name/
└── src/
    ├── fetcher.py
    └── processor.py

# GOOD: all source code in a named package directory
project-name/
├── project_name/
│   ├── __init__.py
│   ├── cli.py
│   ├── processor.py
│   └── errors.py
├── Makefile
└── pyproject.toml
```

The rule: all source files live inside `project_name/`. The project root contains only infrastructure: `Makefile`, `pyproject.toml`, `README.md`, `tests/`, `.venv/`, `make/`.

#### Anti-Pattern 3: Logic in main.py

```python
# BAD: business logic in entry point
def main() -> None:
    args = parse_args()
    data = open(args.input).read()
    result = data.upper()  # logic here!
    print(result)

# GOOD: entry point only handles I/O and calls modules
def main() -> None:
    args = parse_args()
    result = processor.process(args.input)
    print(result)
```

#### Anti-Pattern 4: Bare except or Ignoring Errors

```python
# BAD
try:
    result = process(path)
except:
    pass

# GOOD
try:
    result = process(path)
except ProcessingError as e:
    raise  # or handle explicitly
```

#### Anti-Pattern 5: setup.py Instead of pyproject.toml

```python
# BAD: legacy approach
# setup.py

# GOOD: modern standard
# pyproject.toml
```

#### Anti-Pattern 6: Generic Module Names

```python
# BAD
utils.py
helpers.py
common.py

# GOOD: domain-specific names
processor.py
validator.py
reader.py
```

#### Anti-Pattern 7: Running with System Python

```bash
# BAD: uses system Python, .venv deps not available
python project_name/cli.py
python3 project_name/cli.py

# GOOD: always use .venv's Python
.venv/bin/python -m project_name.cli
make run
```

---

## Monorepo Usage

This skill applies to whichever directory contains `pyproject.toml` — that is the Python component root. The `project_name/` pattern still applies inside each component.

```
monorepo/
└── my-service/               ← component root (pyproject.toml lives here)
    ├── my_service/           ← all source code (project_name/ pattern)
    │   ├── __init__.py
    │   └── cli.py
    ├── tests/
    ├── pyproject.toml
    └── Makefile
```

- `pyproject.toml` and `.venv` live inside `<component>/`, not at the git root
- Run the app from the component root: `cd <component> && .venv/bin/python -m my_service.cli`
- `make/setup.sh` computes `ROOT_DIR` as `$(dirname "$0")/..`, resolving to `<component>/`
- Sub-packages inside `<component>/my_service/` are legitimate domain packages

See **monorepo-skeleton** for the full monorepo layout, root Makefile patterns, and component naming conventions.

---

## Chaining

After completing all steps:

1. **Check the Makefile** — verify it matches the standard in **## Canonical Layout** above (opening lines, .PHONY, help pattern, make/ delegation)
2. **Check the `.gitignore`** — if it is missing the AI Tools section or deviates from the standard, invoke the `gitignore-skeleton` skill
3. **Check the READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill

When creating a complete Python project from scratch, the full workflow involves these skills in order:

1. **`gitignore-skeleton`** — `.gitignore` with Python patterns (.venv, __pycache__, .egg-info)
2. **`readme-skeleton`** — `README.md` with the standard structure (handles `README-PT.md` automatically)

## Related Skills

- **makefile-skeleton** — Generic Makefile structure conventions (opening lines, .PHONY, help pattern)
- **readme-skeleton** — Standard README content and section order
- **gitignore-skeleton** — For bringing `.gitignore` up to the standard after reorganization
- **monorepo-skeleton** — For organizing multi-language monorepos with Python as one component
