---
name: skeleton-scaffold
description: Central orchestrator — detects project type, applies exactly one of go-skeleton, cpp-skeleton, python-skeleton, esp32-skeleton, or monorepo-skeleton, then chains gitignore-skeleton and readme-skeleton.
mode: agent
category: project
shared: true
---

# Skeleton Scaffold

**Single entry point** for scaffolding or reorganizing a repository:

1. Detect the stack using **## Detection** (below).
2. Apply **one** matching language or platform skeleton (that skill owns all layout rules — do not duplicate them here).
3. Chain **gitignore-skeleton**, then **readme-skeleton**.

Other docs (for example [AGENTS.md](AGENTS.md)) list skills but do **not** redefine detection; keep this file as the canonical table.

## Detection

Inspect the current directory (or the user-provided root) for these signals:

| Signal | Skeleton |
|--------|----------|
| `go.mod` | go-skeleton |
| `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt` | python-skeleton |
| `CMakeLists.txt`, `compile_commands.json`, or substantial `*.cpp` / `*.hpp` under `src/` (general C++ application or library) | cpp-skeleton |
| `platformio.ini`, or unambiguous Arduino / ESP8266 / ESP32 embedded layout without the general C++ row above | esp32-skeleton |
| Multiple top-level or clear per-component language roots (e.g. several `go.mod`, mixed stacks, or orchestrator + components) | monorepo-skeleton |
| None of the above | Ask the user |

**Tie-breaks:**

- If several rows could match a **multi-language git root**, prefer **monorepo-skeleton**, then apply the per-component skeleton inside each component directory as that component’s skill describes.
- If both **cpp-skeleton** and **esp32-skeleton** could match, prefer **esp32-skeleton** only when `platformio.ini` or embedded intent is clear; otherwise prefer **cpp-skeleton**.

## Workflow

1. **Detect** using the table above.
2. **Confirm** with the user (example: *"Detected: C++. Applying cpp-skeleton — proceed?"*).
3. **Apply** the chosen skeleton — read and follow that skill’s full instructions.
4. **Apply gitignore-skeleton** — create or update `.gitignore`.
5. **Apply readme-skeleton** — create or update `README.md` (and bilingual files per that skill).

## Notes

- If detection is ambiguous, ask the user to pick a skeleton.
- Never apply more than one **language/platform** skeleton to the same component directory at once.
- In monorepos: fix each component with its skeleton first; use **monorepo-skeleton** for the git root orchestrator layout.

## Delegated and chained skills

| Phase | Skills |
|-------|--------|
| Layout (pick one) | **go-skeleton**, **cpp-skeleton**, **python-skeleton**, **esp32-skeleton**, **monorepo-skeleton** |
| Always after layout | **gitignore-skeleton**, then **readme-skeleton** |

## Related Skills

- **sync-skills** — Run after changes to propagate corrected skills to all destinations.
