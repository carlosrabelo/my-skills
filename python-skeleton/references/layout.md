# Python Project — Standard Layout

Canonical target structure for all Python projects. Source files live directly at the project root, the virtual environment stays as `.venv`, and `pyproject.toml` defines the project.

**Key principles**:
- `.venv` is always at the project root — a sibling of the source files
- `pyproject.toml` is the single source of truth for metadata and dependencies
- Source files live directly at the project root — sub-packages only when a true module is needed
- The main entry point can be named anything (`main.py`, `fetcher.py`, `cli.py`, etc.) — use a domain-specific name, not necessarily `main.py`
- Applications are run with `python -m <module>` using `.venv/bin/python`
- Python version: **3.12**

## Standard Project Layout

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
├── fetcher.py                    ← Main entry point (domain-named, run via python -m)
├── processor.py                  ← Core logic module
├── errors.py                     ← Custom exception types
├── models/                       ← Sub-package ONLY if there are multiple related modules
│   ├── __init__.py
│   └── item.py
│
└── tests/                        ← All tests
    ├── conftest.py               ← Pytest fixtures and configuration
    ├── test_fetcher.py
    ├── test_processor.py
    └── models/                   ← Mirror sub-packages if they exist
        └── test_item.py
```

> The main entry point is named after what it does — `fetcher.py`, `cli.py`, `converter.py`, `scraper.py`, etc. Only use `main.py` if the project has no better domain name for the entry point.

### When to Create a Sub-Package

Only create a sub-folder at the project root when:
- It groups **multiple files** that form a cohesive module (e.g., `models/`, `api/`)
- It needs an `__init__.py` to be importable as a package
- A single file would become too large or have mixed concerns

**Never** create `utils/`, `helpers/`, or `common/` folders — give the module a domain-specific name.

---

## `pyproject.toml` — Project Definition

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
project-name = "fetcher:main"   # ← use actual module name and entry function

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

> ⚠️ For script-based projects, **do not add `packages.find`** — loose files at the project root don't need it.

---

## `.venv/` — Virtual Environment

**Location**: Always at project root. Never commit.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
# or, for script-only projects:
pip install ".[dev]"
```

---

## Makefile

Single entry point for all operations. Delegates to `make/` scripts.

```makefile
MAKEFLAGS += --no-print-directory

PY_FILES := $(wildcard *.py)

.PHONY: setup test lint fmt typecheck clean install run help

setup:
	./make/setup.sh

test:
	./make/test.sh

lint:
	./make/lint.sh

fmt:
	.venv/bin/ruff format $(PY_FILES) tests/

typecheck:
	.venv/bin/mypy $(PY_FILES)

clean:
	rm -rf dist/ build/ *.egg-info
	find . -type d -name __pycache__ -exec rm -rf {} +
	find . -type f -name "*.pyc" -delete

run:
	.venv/bin/python -m fetcher   # ← use actual module name; append args via: make run ARGS="--help"

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

## make/ — Shell Scripts

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

"$ROOT_DIR/.venv/bin/ruff" check "$ROOT_DIR"/*.py tests/
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

## .gitignore

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

## Monorepo Usage

This skill applies to whichever directory contains `pyproject.toml` — that is the Python project root.

- `pyproject.toml` and `.venv` live inside `<component>/`, not at the git root
- Run the app from the component root: `cd <component> && .venv/bin/python -m <module>`
- Sub-packages inside `<component>/` are legitimate domain packages, not the `src/` anti-pattern
