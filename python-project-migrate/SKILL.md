---
name: python-project-migrate
description: Reorganize existing Python projects to match the standard flat root layout (source files at root, sub-packages only when justified, tests/). Handles migration from src/ layout, monolithic code, and missing structure. Invoke manually with /python-project-migrate.
mode: manual
category: python
shared: true
---

# Python Project Migrate

Reorganize existing Python projects to match the standard structure defined in `python-project-create`. Invoke manually with `/python-project-migrate` when a project needs structural alignment.

## Overview

This skill transforms messy or legacy Python projects into the standard layout:

```
project-name/
├── Makefile
├── pyproject.toml
├── main.py           ← Entry point with shebag (≤50 lines)
├── processor.py      ← Domain-specific modules at root
├── errors.py         ← Custom exception types
├── run/              ← Automation scripts
│   ├── install.sh
│   └── uninstall.sh
└── tests/
    ├── conftest.py
    └── test_processor.py
```

---

## Before You Start

### Checklist

- ✅ All changes committed (`git status` clean)
- ✅ Tests passing (`make test`)
- ✅ Understand what the project does
- ✅ Identify what exists and what's missing

### Diagnose the Project

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

---

## Reorganization Scenarios

### Scenario 1: Migrate from src/ Layout

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
PY_FILES := $(wildcard *.py)

fmt:
	.venv/bin/ruff format $(PY_FILES) tests/

typecheck:
	.venv/bin/mypy $(PY_FILES)
```

5. **Update `run/lint.sh`** and **`run/install.sh`**:
```bash
# Old: "$ROOT_DIR/.venv/bin/ruff" check src/ tests/
# New:
"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR"/*.py tests/

# Old: cp "$ROOT_DIR/src/$SCRIPT_NAME" "$INSTALL_DIR/$BINARY_NAME"
# New:
cp "$ROOT_DIR/$SCRIPT_NAME" "$INSTALL_DIR/$BINARY_NAME"
```

6. **Remove `sys.path` hacks from tests** (no longer needed):
```python
# Delete these lines from test files:
# import sys
# import os
# sys.path.insert(0, os.path.join(os.path.dirname(__file__), "..", "src"))
```

7. **Verify**:
```bash
make test
make lint
make typecheck
```

---

### Scenario 2: Monolithic main.py

**Symptom**: `main.py` is 200+ lines with business logic mixed into the entry point.

**Steps**:

1. **Identify concerns** in main.py:
   - CLI/argument parsing
   - Business/processing logic
   - Error types
   - Output formatting

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

### Scenario 3: Oversized Module

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

### Scenario 4: Generic Module Names

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

### Scenario 5: Duplicated Code Across Modules

**Symptom**: Same function or logic copy-pasted in multiple files.

**Steps**:

1. **Compare the duplicated code**:
```bash
grep -A 10 "def validate_path" convert.py
grep -A 10 "def validate_path" export.py
```

2. **Create a shared module** at project root (e.g., `validation.py`)

3. **Move the common implementation** to the shared module

4. **Update all importing files** to use the shared module

5. **Delete duplicate code** from original files

6. **Verify**:
```bash
make test
```

---

### Scenario 6: Flatten Unnecessary Sub-Package

**Symptom**: A sub-package directory contains only one file.

**Before**:
```
project-name/
└── models/
    ├── __init__.py
    └── item.py       ← only one file
```

**Steps**:

1. **Move file to project root**:
```bash
mv models/item.py item.py
rm models/__init__.py
rmdir models/
```

2. **Update all imports**:
```bash
grep -r "from models" *.py tests/
# Old: from models.item import Item
# New: from item import Item
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

### Scenario 7: Introduce Sub-Package When Justified

**Symptom**: 5+ files of the same domain are hard to distinguish from unrelated logic files.

**Steps**:

1. **Create the sub-package**:
```bash
mkdir -p models
touch models/__init__.py
```

2. **Move the files** and update imports:
```bash
mv user.py item.py order.py models/
# Old: from user import User
# New: from models.user import User
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

### Scenario 8: Add Proper Test Fixtures

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

### Scenario 9: Migrate requirements.txt to pyproject.toml

**Symptom**: Project uses `requirements.txt` for dependencies instead of `pyproject.toml`.

**Steps**:

1. **Create `pyproject.toml`** at the project root with dependencies from `requirements.txt`:
```toml
[build-system]
requires = ["setuptools>=68", "wheel"]
build-backend = "setuptools.backends.legacy:build"

[project]
name = "project-name"
version = "0.1.0"
description = "Short description"
requires-python = ">=3.12"
dependencies = [
    # Copy runtime deps from requirements.txt
    "requests>=2.31",
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

2. **Update `run/setup.sh`** to install from `pyproject.toml` instead of `requirements.txt`:
```bash
# Old:
"$ROOT_DIR/.venv/bin/pip" install -r "$ROOT_DIR/requirements.txt"

# New:
"$ROOT_DIR/.venv/bin/pip" install -e "$ROOT_DIR[dev]" --quiet
```

3. **Keep `requirements.txt` as a lockfile** (or remove it):
```bash
# Option A: keep as lockfile (pinned versions for reproducible installs)
pip freeze > requirements.txt

# Option B: remove it (pyproject.toml is now the source of truth)
rm requirements.txt
```

4. **Verify**:
```bash
rm -rf .venv
make setup    # rebuilds .venv from pyproject.toml
make test
```

---

## Reorganization Checklist

### Before
- [ ] `git status` is clean
- [ ] All tests pass
- [ ] Understand current structure

### During
- [ ] Move files to correct locations
- [ ] Update all imports (moved code + consuming code)
- [ ] Remove `sys.path` hacks if present
- [ ] Tests still pass after each move

### After
- [ ] `make test` passes
- [ ] `make typecheck` passes
- [ ] `make lint` passes
- [ ] `main.py` ≤ 50 lines
- [ ] No `src/` directory
- [ ] No `utils.py` or `helpers.py`
- [ ] No sub-packages wrapping single files
- [ ] Commit with clear message

---

## Rules

- **Never mix reorganization with logic changes** — reorganize first, then modify behavior in a separate commit
- **Move one file at a time** — move, update imports, verify tests, repeat
- **No generic module names** — `utils.py`, `helpers.py`, `common.py` must be renamed to domain-specific names
- **No sub-packages for single files** — flatten to a root-level module instead
- **main.py must be thin** — ≤50 lines, CLI parsing + delegation only
- **No `src/` directory** — source files belong at project root

---

## Monorepo Usage

This skill applies to whichever directory contains `pyproject.toml` — that is the Python project root.

When migrating a Python component inside a monorepo:

- Treat `<component>/` as the project root throughout all scenarios. `pyproject.toml`, `.venv`, `Makefile`, `run/`, and all source files live there.
- The self-relaunching shebag works unchanged: `Path(__file__).resolve().parent` resolves to `<component>/` — no path adjustments needed.
- `run/setup.sh` computes `ROOT_DIR` as `$(dirname "$0")/..`, resolving to `<component>/`. `.venv` is created there, isolated from the git root.
- Sub-packages inside `<component>/` are legitimate domain packages — not Anti-Pattern 2.

See **monorepo-project-create** for the full monorepo layout and root Makefile patterns.

---

## Encadeamento

Após concluir todos os passos acima:

1. **Verifique o Makefile** — se existir mas não seguir o padrão de `makefile-create`, invoque a skill `makefile-migrate`
2. **Verifique os READMEs** — se `README.md` ou `README-PT.md` precisarem de atualização, invoque a skill `readme-migrate`
3. **Invoque `git-commit-suggest`** para preparar o commit com as mudanças feitas

---

## Related Skills

- **python-project-create** — The target structure this skill reorganizes toward
- **readme-bilingual-sync** — Update docs after significant structural changes
- **monorepo-project-create** — For organizing the git root when Python is one component among many
- **monorepo-project-migrate** — For migrating the repo itself into a monorepo layout
