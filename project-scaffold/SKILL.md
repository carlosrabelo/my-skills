---
name: project-scaffold
description: Detects project type from existing files and applies the appropriate skeleton (go, python, esp32, monorepo), then chains gitignore-skeleton and readme-skeleton.
mode: agent
category: project
shared: true
---

# Project Scaffold

Single entry point for scaffolding or reorganizing any project. Detects the project type automatically and delegates to the appropriate skeleton, then completes the setup with gitignore and readme.

## Detection

Inspect the current directory for these signals to determine project type:

| Signal | Skeleton |
|--------|---------|
| `go.mod` | go-skeleton |
| `pyproject.toml`, `setup.py`, `setup.cfg`, `requirements.txt` | python-skeleton |
| `platformio.ini` or `src/main.cpp` (no `go.mod` / `pyproject.toml`) | esp32-skeleton |
| Multiple subdirectories each with their own language files or `Makefile` | monorepo-skeleton |
| None of the above | Ask the user |

When multiple signals are present (e.g. a monorepo with Go and Python components), prefer **monorepo-skeleton**.

## Workflow

1. **Detect** the project type using the table above.
2. **Confirm** with the user: *"Detected: Go project. Applying go-skeleton — proceed?"*
3. **Apply** the detected skeleton (read and follow that skill's full instructions).
4. **Apply gitignore-skeleton** — create or update `.gitignore`.
5. **Apply readme-skeleton** — create or update `README.md`.

## Notes

- If detection is ambiguous, ask the user to choose the skeleton.
- Never apply multiple language skeletons to the same component directory.
- In monorepos, apply the per-component skeleton to each component subdirectory first, then apply monorepo-skeleton for the root orchestrator.

## Related Skills

- **go-skeleton** — Applied for Go projects
- **python-skeleton** — Applied for Python projects
- **esp32-skeleton** — Applied for ESP32/ESP8266 PlatformIO projects
- **monorepo-skeleton** — Applied for multi-component repositories
- **gitignore-skeleton** — Always applied after the language skeleton
- **readme-skeleton** — Always applied after gitignore-skeleton
