# Python Project — Patterns

Running the application, testing patterns, error handling, and anti-patterns for Python projects.

## Running the Application

Applications are run with `python -m <module>` using the project's `.venv`:

```bash
# Standard way to run
.venv/bin/python -m fetcher
.venv/bin/python -m fetcher --help

# Or via Makefile
make run

# After activating the venv
source .venv/bin/activate
python -m fetcher --help
```

The entry point module must have an `if __name__ == "__main__"` block:

```python
"""Command-line interface for project-name."""

import argparse
import sys

from processor import process
from errors import ProcessingError, ValidationError


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

### Installing as a Command

Add `[project.scripts]` to `pyproject.toml` to make the tool available as a named command after `pip install`:

```toml
[project.scripts]
project-name = "fetcher:main"   # ← "module:function"
```

After `make setup` (which runs `pip install -e .`), the command is available at `.venv/bin/project-name`. The `make/install.sh` script copies it to `~/.local/bin/`.

---

## Testing Patterns

### Structure

Tests live in `tests/`, with one test file per source file:

```
tests/
├── conftest.py               ← Shared fixtures
├── test_fetcher.py
├── test_processor.py
└── models/                   ← Mirror sub-packages if they exist
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

With `pythonpath = ["."]` in `pyproject.toml`, imports work directly — no `sys.path` hack needed:

```python
# tests/test_processor.py
"""Tests for processor."""

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

### Running Tests

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

## Anti-Patterns: What NOT to Do

### ❌ Anti-Pattern 1: Committing .venv

```bash
# ❌ BAD
git add .venv/

# ✅ GOOD: .venv in .gitignore, reproducible via make/setup.sh
```

### ❌ Anti-Pattern 2: Wrapping Source Files in a Package

This is the **most common mistake** when following setuptools docs or other project templates.

```
# ❌ BAD: named sub-package wrapping all source files
project-name/
└── project_name/
    ├── __init__.py
    └── cli.py

# ❌ BAD: src/ directory wrapping source files (overkill for scripts)
src/
├── fetcher.py
├── processor.py
└── errors.py

# ✅ GOOD: files live directly at the project root
project-name/
├── fetcher.py
├── processor.py
└── errors.py
```

The rule: source files live directly at the project root. No `src/` wrapper, no `project_name/` wrapper.

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
utils.py
helpers.py
common.py

# ✅ GOOD: domain-specific names
processor.py
validator.py
reader.py
```

### ❌ Anti-Pattern 7: Running with System Python

```bash
# ❌ BAD: uses system Python, .venv deps not available
python fetcher.py
python3 fetcher.py

# ✅ GOOD: always use .venv's Python
.venv/bin/python -m fetcher
make run
```
