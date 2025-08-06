---
name: cpp-skeleton
description: Standard C++ project layout (repo root + projectname/ component, src/tests/lib, Makefile + .make/ scripts, g++, Catch2). Creates from scratch or reorganizes existing projects. Modeled on go-skeleton conventions and typical CLI/library structure.
mode: agent
category: project
shared: true
---

# C++ Skeleton

Unified skill for organizing C++ projects that use a **root orchestrator Makefile** plus a **named subdirectory** (`projectname/`) holding all source. Automation lives in **`.make/`** at the repository root; build outputs go to **`bin/`** at the repository root. This matches the pattern used by projects such as **Mojave**: root `Makefile` delegates with `$(MAKE) -C projectname`, component `Makefile` invokes `../.make/*.sh`.

## Layout

> **Infrastructure at the repository root. All C++ sources under `projectname/src/`. Tests under `projectname/tests/`.**

```
project-name/                    ← git root (kebab-case repo folder)
├── Makefile                     ← orchestrator: $(MAKE) -C projectname
├── .make/                       ← shell scripts (see below)
├── bin/                         ← artifacts (bin/* gitignored; bin/.gitkeep)
├── README.md
└── projectname/                 ← component root (hyphens removed from project-name)
    ├── Makefile                 ← delegates to ../.make/*.sh
    ├── src/
    │   ├── main.cpp             ← entry point (thin: argv + dispatch)
    │   ├── core/                ← domain code (headers + .cpp as needed)
    │   └── ...
    ├── tests/
    │   ├── test_main.cpp        ← Catch2 test runner / registration
    │   └── test_*.cpp
    └── lib/                     ← optional vendored deps (e.g. Catch2 header)
```

**Naming**:

- **`project-name`**: repository directory (often kebab-case).
- **`projectname`**: inner component directory — ASCII only, hyphens removed (`my-app` → `myapp`, `mojave` → `mojave`).
- **Binary basename**: defaults to `projectname` (executable appears as `bin/projectname`).

## Conventions

- **Standard**: **C++20** (adjust `-std=` if the project requires another revision).
- **Compiler**: `g++` by default; honor **`CXX`** and extend flags via script variables when cross-compiling.
- **Includes**: use **`#include <module/file.hpp>`** with **`-I projectname/src`** so includes stay stable when moving translation units.
- **Tests**: **Catch2** single-header vendored under **`projectname/lib/catch.hpp`** via `make setup` (download script); compile all `tests/*.cpp` into one test binary **`bin/projectname-tests`** at repo root.
- **Quality**: **`clang-format`** for formatting, **`cppcheck`** for static analysis, **`g++ -fsyntax-only`** for fast syntax checks — mirroring a typical Linux developer workflow.

## Context Detection

1. **Signals** (any suggest C++): `CMakeLists.txt`, `compile_commands.json`, many `*.cpp` / `*.hpp` / `*.cc`, `meson.build`, root or nested `Makefile` compiling with `g++`/`clang++`.
2. **Establish root**: if the user points at a subdirectory that contains `src/main.cpp` and a component `Makefile`, that subdirectory may be **`projectname/`** — the git root is usually one level above (where `.make/` and orchestrator `Makefile` live). Confirm with `git rev-parse --show-toplevel`.
3. **If structure already matches** this skill's canonical layout → small fixes only (scripts, `.gitignore`, README).
4. **If structure is flat or ad hoc** → follow **## Migrating an Existing C++ Project**.
5. **Green field** → follow **## Creating from Scratch**.

---

## Migrating an Existing C++ Project

### Mandatory Checklist

#### Before

- [ ] `git status` clean or user accepts a WIP branch
- [ ] Project builds and tests pass with current flow
- [ ] Map current directories to `projectname/src`, `projectname/tests`

#### During

- [ ] Introduce `projectname/` (or rename existing) consistent with naming rules
- [ ] Move translation units and headers; fix `#include` paths
- [ ] Root `Makefile` only delegates; **`.make/`** scripts contain compiler logic
- [ ] `touch bin/.gitkeep` if using `bin/*` / `!bin/.gitkeep` ignore pattern

#### After

- [ ] `make build` produces `bin/<binary>`
- [ ] `make test` passes
- [ ] `make quality` (or fmt + lint + test) passes where tools are installed
- [ ] **gitignore-skeleton** includes **CPP** section (or equivalent entries)
- [ ] No duplicate `Makefile` logic at root beyond orchestration

### Migration Scenarios

#### Scenario A: Flat sources at repository root

**Before**: `main.cpp`, headers, and `Makefile` mixed at top level.

**Steps**:

1. Pick **`projectname`** from the repo name (hyphens removed).
2. `mkdir -p projectname/src projectname/tests bin && touch bin/.gitkeep`
3. Move sources: `mv main.cpp projectname/src/` (and other `.cpp` / module dirs into `projectname/src/`).
4. Replace root **`Makefile`** with an orchestrator that only runs `$(MAKE) -C projectname`.
5. Add **`projectname/Makefile`** and **`.make/`** scripts per **## Canonical Makefile and Scripts**.
6. Update includes to `<...>` form with `-I projectname/src`.

#### Scenario B: Everything under `src/` at root (no inner component dir)

**Before**: `./src`, `./Makefile`, no `projectname/` child.

**Steps**:

1. Create **`projectname/`**, move **`src/`** → **`projectname/src/`**.
2. Add orchestrator root **`Makefile`** with `$(MAKE) -C projectname`.
3. Move **`.make/`** (if any) to repo root; fix **`ROOT_DIR`** / **`REPO_ROOT`** in scripts to match **## Script Environment Variables**.

#### Scenario C: Adding tests from scratch

1. `make setup` pattern: download **`catch.hpp`** into `projectname/lib/` (or vendor another test framework — update **`test.sh`** accordingly).
2. Add **`tests/test_main.cpp`** and **`tests/test_*.cpp`**; ensure **`test.sh`** compiles every **`tests/*.cpp`** in sorted order into **`bin/projectname-tests`**.

---

## Creating from Scratch

1. Create layout from **## Layout**; use **`mkdir -p`** and **`touch bin/.gitkeep`**.
2. Add **`README.md`** (or invoke **readme-skeleton**).
3. Add **`.gitignore`** with **CPP** section from **gitignore-skeleton**.
4. Populate **`projectname/src/main.cpp`** (minimal).
5. Implement **`.make/`** scripts and both **Makefiles** from **## Canonical Makefile and Scripts**.
6. Run **`make setup`**, **`make build`**, **`make test`**.

---

## Canonical Makefile and Scripts

### Root `Makefile` (orchestrator only)

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build check clean fmt help lint quality run setup test

help: ## Show available targets
	@$(MAKE) -C projectname help

setup: ## Download dependencies (e.g. Catch2 header)
	@$(MAKE) -C projectname setup

build: ## Compile binary
	@$(MAKE) -C projectname build

run: ## Run binary
	@$(MAKE) -C projectname run

test: ## Build and run tests
	@$(MAKE) -C projectname test

check: ## Syntax-check sources
	@$(MAKE) -C projectname check

lint: ## Run cppcheck
	@$(MAKE) -C projectname lint

fmt: ## Format with clang-format
	@$(MAKE) -C projectname fmt

quality: ## fmt + lint + test
	@$(MAKE) -C projectname quality

clean: ## Remove build artifacts
	@$(MAKE) -C projectname clean
```

Replace **`projectname`** with the actual directory name.

### Component `projectname/Makefile`

```makefile
MAKEFLAGS += --no-print-directory

.DEFAULT_GOAL := help

.PHONY: build check clean fmt help lint quality run setup test

help: ## Show available targets
	@echo "projectname - Available targets"
	@echo ""
	@grep -hE '^[a-zA-Z_-]+:.*## ' $(MAKEFILE_LIST) \
		| sort \
		| awk 'BEGIN {FS = ":.*## "} {printf "  %-15s %s\n", $$1, $$2}'

setup: ## Download dependencies
	@../.make/setup.sh

build: ## Compile binary
	@../.make/build.sh

run: ## Run binary
	@../.make/run.sh

test: ## Compile and run tests
	@../.make/test.sh

check: ## Syntax-check all source files
	@../.make/check.sh

lint: ## Run cppcheck
	@../.make/lint.sh

fmt: ## Format source
	@../.make/fmt.sh

quality: ## All quality checks
	@../.make/quality.sh

clean: ## Remove build artifacts
	@rm -rf ../bin/
```

Paths assume **`projectname/Makefile`** is one level below repo root; **`.make/`** lives next to the orchestrator **`Makefile`**.

### Script environment variables

Every script starts with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

Use:

- **`ROOT_DIR`**: absolute path to **`projectname/`** (component root):  
  `ROOT_DIR="$(cd "$(dirname "$0")/../projectname" && pwd)"`  
  (replace **`projectname`**).
- **`REPO_ROOT`**: repository root (parent of **`projectname/`**):  
  `REPO_ROOT="$(cd "$(dirname "$0")/.." && pwd)"`
- **`SRC_DIR="$ROOT_DIR/src"`**, **`TEST_DIR="$ROOT_DIR/tests"`**, **`LIB_DIR="$ROOT_DIR/lib"`**, **`BUILD_DIR="$REPO_ROOT/bin"`**
- **`BINARY_NAME`**: executable basename (default **`projectname`**)
- **`CXX="${CXX:-g++}"`**
- **`CXXFLAGS`**: at minimum `-std=c++20 -Wall -Wextra -Wpedantic -I$SRC_DIR` (add **`-I$LIB_DIR`** for tests)

### `.make/build.sh` (template)

Compile the main program; extend with more **`*.cpp`** as translation units appear.

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/../projectname" && pwd)"
REPO_ROOT="$(cd "$(dirname "$0")/.." && pwd)"
SRC_DIR="$ROOT_DIR/src"
BUILD_DIR="$REPO_ROOT/bin"
BINARY_NAME="${BINARY_NAME:-projectname}"
CXX="${CXX:-g++}"
CXXFLAGS="-std=c++20 -Wall -Wextra -Wpedantic -I$SRC_DIR"

mkdir -p "$BUILD_DIR"
echo "Building $BINARY_NAME..."
# Start with main TU; add more sources as needed:
$CXX $CXXFLAGS "$SRC_DIR/main.cpp" -o "$BUILD_DIR/$BINARY_NAME"
echo "Done: $BUILD_DIR/$BINARY_NAME"
```

### `.make/run.sh` (template)

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_ROOT="$(cd "$(dirname "$0")/.." && pwd)"
BUILD_DIR="$REPO_ROOT/bin"
BINARY_NAME="${BINARY_NAME:-projectname}"

if [ ! -f "$BUILD_DIR/$BINARY_NAME" ]; then
    echo "Binary not found. Run 'make build' first."
    exit 1
fi

exec "$BUILD_DIR/$BINARY_NAME" "$@"
```

### `.make/test.sh` (template)

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/../projectname" && pwd)"
REPO_ROOT="$(cd "$(dirname "$0")/.." && pwd)"
SRC_DIR="$ROOT_DIR/src"
TEST_DIR="$ROOT_DIR/tests"
LIB_DIR="$ROOT_DIR/lib"
BUILD_DIR="$REPO_ROOT/bin"
BINARY_NAME="${BINARY_NAME:-projectname}"
CXX="${CXX:-g++}"
CXXFLAGS="-std=c++20 -Wall -Wextra -Wpedantic -I$SRC_DIR -I$LIB_DIR"

mkdir -p "$BUILD_DIR"
mapfile -t TEST_SRCS < <(find "$TEST_DIR" -maxdepth 1 -name '*.cpp' -print | LC_ALL=C sort)
$CXX $CXXFLAGS "${TEST_SRCS[@]}" -o "$BUILD_DIR/${BINARY_NAME}-tests"
echo "Running tests..."
exec "$BUILD_DIR/${BINARY_NAME}-tests" "$@"
```

### `.make/setup.sh` (Catch2 header)

Download **`catch.hpp`** only if missing; pin a version for reproducibility.

### `.make/check.sh`

Run **`$CXX -fsyntax-only`** over **`src/**/*.cpp`**.

### `.make/lint.sh`

Require **`cppcheck`**, analyze **`SRC_DIR`** with **`--std=c++20`** (tune suppressions for third-party includes).

### `.make/fmt.sh`

Use **`clang-format -i`** on **`src/`** and **`tests/`** with correct **`find`** grouping:

```bash
find "$SRC_DIR" "$TEST_DIR" \( -name '*.cpp' -o -name '*.hpp' \) -print0 | xargs -0 clang-format -i
```

### `.make/quality.sh`

Sequence: format (if **`clang-format`** exists), **`cppcheck`** (if installed), then compile and run tests — fail on first hard error where tools exist.

---

## Monorepo / CMake

- **Monorepo**: each C++ component gets its own **`projectname/`** subtree with this layout; the git root **`Makefile`** delegates per component (see **monorepo-skeleton**). **If several C++ components share one repository**, set **`BUILD_DIR`** to a per-component output directory (for example **`"$ROOT_DIR/bin"`** inside that component) so binaries from different components do not overwrite each other — a single shared **`$REPO_ROOT/bin`** is only appropriate when there is at most one C++ component using it.
- **CMake**-primary repos are out of scope for this skeleton; if the user standardizes on CMake, prefer documenting **`cmake --build`** targets and optionally a thin **`Makefile`** facade — do not fight the existing generator without an explicit migration goal.

---

## Related Skills

- **go-skeleton** — Parallel “named subfolder + root orchestrator” mental model
- **makefile-skeleton** — Make opening lines, `.PHONY`, help target, `.make/` delegation
- **gitignore-skeleton** — Add **CPP** section for `bin/*`, `obj/`, artifacts
- **readme-skeleton** — Documentation structure after layout is stable
- **skeleton-scaffold** — Use when the repo is unknown or multi-stack; detection table lives there — this skill applies once C++ is chosen
