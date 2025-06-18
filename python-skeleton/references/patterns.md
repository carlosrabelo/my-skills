# Python Project — Patterns

Shebags, testing patterns, error handling, and anti-patterns for Python projects.

## Shebags and .venv

Python scripts at the project root meant to be run directly (e.g. `./fetcher.py`) must point to the right interpreter. On Linux, shebags **must be absolute paths** — relative paths like `#!.venv/bin/python` are not supported by the kernel.

The solution is the **self-relaunching pattern**: the script starts with the system Python, detects `.venv` in the same directory, and re-executes itself using the `.venv` interpreter if not already running inside it.

### The Self-Relaunching Pattern

Scripts live at the project root, so `.venv` is a sibling:

```python
#!/usr/bin/env python3
"""Script description."""

# Auto-switch to .venv (same directory as the script)
import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent / ".venv" / "bin" / "python"

if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# --- script starts here, guaranteed to be running inside .venv ---

import requests  # safe: .venv deps are available
```

**How it works**:
1. System Python runs the script first
2. `Path(__file__).resolve().parent` resolves to the project root (where the script lives)
3. If `.venv/bin/python` exists there and is not the current interpreter, `os.execv` replaces the current process — no subprocess, no overhead
4. The script re-runs from the top, this time already inside `.venv`, so the check is skipped

### Script Template

```python
#!/usr/bin/env python3
"""One-line description of what this script does."""

import os
import sys
from pathlib import Path

# Auto-switch to .venv
_venv_python = Path(__file__).resolve().parent / ".venv" / "bin" / "python"
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

### Multiple Scripts

Each script at the project root is self-contained. Copy the relauncher block at the top of every script that needs `.venv` deps:

```python
#!/usr/bin/env python3
"""Another script."""

import os
import sys
from pathlib import Path

_venv_python = Path(__file__).resolve().parent / ".venv" / "bin" / "python"
if _venv_python.exists() and Path(sys.executable).resolve() != _venv_python.resolve():
    os.execv(str(_venv_python), [str(_venv_python)] + sys.argv)

# script-specific code below
from processor import process  # relative import from project root
```

### Why Not Other Approaches

| Approach | Problem |
|----------|---------|
| `#!.venv/bin/python` | Relative shebags not supported on Linux |
| `#!/usr/bin/env python3` alone | Uses system Python, deps not available |
| Activate venv manually | Requires user to remember every time |
| Bash wrapper script | Extra file, more complexity |
| `#!/absolute/path/.venv/bin/python` | Path changes per machine, not portable |

---

## Testing Patterns

### Structure

Tests live in `tests/`, with one test file per source file:

```
tests/
├── conftest.py               ← Shared fixtures
├── test_main.py
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

### ❌ Anti-Pattern 7: Relative shebag

```python
# ❌ BAD: not supported by Linux kernel
#!/.venv/bin/python
#!.venv/bin/python

# ✅ GOOD: self-relaunching pattern with .venv
#!/usr/bin/env python3
# + the os.execv relauncher block
```
