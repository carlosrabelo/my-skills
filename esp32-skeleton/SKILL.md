---
name: esp32-skeleton
description: Standard ESP32/ESP8266 Arduino project structure (PlatformIO, src/, make/ scripts). Creates from scratch or reorganizes existing projects.
mode: agent
category: project
shared: true
---

# ESP32 Skeleton

Unified skill for organizing ESP32/ESP8266 Arduino projects using PlatformIO. Handles both new projects and reorganization of existing ones.

## Context Detection

Before starting, determine the context:

1. **Check for `platformio.ini`** in the current directory (or the target directory).

2. **If `platformio.ini` exists** → this is an existing project that needs reorganization. Read `references/migrate.md`.

3. **If `platformio.ini` does not exist** → this is a new project being created from scratch. Read `references/create.md`.

4. **Always read `references/layout.md`** — it is the canonical target structure for both flows.

5. **Read `references/platformio.md`** for all PlatformIO configuration patterns (single-board, multi-board, multi-platform with ESP32+ESP8266).

## Reference Files

| File | When to read |
|------|-------------|
| `references/layout.md` | Always — canonical structure |
| `references/create.md` | Creating a new project from scratch |
| `references/migrate.md` | Reorganizing an existing PlatformIO project |
| `references/platformio.md` | PlatformIO configuration: single-board, multi-board, multi-platform |

## Complexity Tiers

Projects fall into one of three tiers — choose based on what the user describes:

| Tier | When | Examples |
|------|------|---------|
| **Simple** | Single board, no libraries, no secrets | LED blink, basic GPIO |
| **Standard** | Single board, external libraries, secret credentials | Sensor reading with WiFi |
| **Advanced** | Multiple boards or platforms, tests, build variants | Mining firmware, proxy with web UI |

## Chaining

When creating a complete project from scratch, the full workflow involves these skills in order:

1. **`esp32-skeleton`** — create the PlatformIO structure and Makefile
2. **`gitignore-skeleton`** — `.gitignore` with PlatformIO artifacts, secrets, and editor dirs
3. **`readme-skeleton`** — `README.md` with the standard structure

## Related Skills

- **gitignore-skeleton** — Standard `.gitignore` adapted for PlatformIO projects
- **readme-skeleton** — Standard README content and bilingual conventions
- **makefile-skeleton** — Makefile patterns (the ESP32 Makefile follows similar conventions)
