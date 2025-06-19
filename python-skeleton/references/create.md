# Python Project — Creating from Scratch

Instructions for creating a new Python project following the standard layout. Read `layout.md` for the canonical structure before applying these instructions.

## Project Layout

### Files at Project Root

The main files of the project live directly at the project root. The entry point is named after what it does:

```
project-name/
├── fetcher.py     ← Entry point: has shebag + arg parsing (thin). Name matches the domain.
├── processor.py   ← Core business logic
└── errors.py      ← Custom exception types
```

Use `main.py` only if no better name exists. Good names: `fetcher.py`, `converter.py`, `scraper.py`, `cli.py`, `importer.py`.

### When to Add a Sub-Package

Only add a sub-folder when it groups **multiple files** forming a cohesive module:

```
project-name/
├── main.py
├── processor.py
├── errors.py
└── models/        ← Sub-package: multiple model types together
    ├── __init__.py
    ├── user.py
    └── item.py
```

Do **not** create `utils/`, `helpers/`, `common/` — use domain-specific names.

---

## Entry Point Template

```python
"""Command-line interface for project-name."""

import argparse
import sys

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

```bash
.venv/bin/python -m fetcher --help
```

---

## `processor.py` — Business Logic

```python
# processor.py
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

---

## `errors.py` — Custom Exceptions

```python
# errors.py
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

## Key Principles

### 1. Virtual Environment
- Always `.venv` at project root
- Scripts at root use `parent / ".venv"` to find it
- Reproducible via `make/setup.sh`

### 2. Dependencies
- Runtime deps in `[project.dependencies]`
- Dev deps in `[project.optional-dependencies] dev`
- Always pin minimum versions (`>=`)

### 3. File Organization
- Source files directly at the project root — **never create a wrapper directory** (`src/` or `project_name/`), this is the most common mistake
- Sub-packages only when multiple files form a true module (e.g. `models/`)
- Tests in `tests/`, mirroring project root structure

### 4. Testing
- Use `pythonpath = ["."]` in `pyproject.toml` — no `sys.path` manipulation needed
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
| Add new entry point | `domain-name.py` at project root | Thin: parse args, call modules |
| Add business logic | `processor.py` or domain-named file at root | Keep thin layers |
| Add custom exception | `errors.py` at root | Extend `ProjectError` |
| Add multiple related types | `models/` at root | Only if multiple files are needed |
| Add test | `tests/` mirroring project root | One test file per source file |
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

# Run the application (use the actual module name)
.venv/bin/python -m fetcher --help
make run

# Before committing
make lint
make test

# Install locally (requires [project.scripts] in pyproject.toml)
make install
project-name --help
```
