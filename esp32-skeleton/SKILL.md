---
name: esp32-skeleton
description: Standard ESP32/ESP8266 Arduino project structure (PlatformIO, src/, make/ scripts). Creates from scratch or reorganizes existing projects.
mode: agent
category: project
shared: true
---

# ESP32 Skeleton

Unified skill for organizing ESP32/ESP8266 Arduino projects using PlatformIO. Handles both new projects and reorganization of existing ones.

## CRITICAL — Variable Naming (READ THIS FIRST)

The Makefile and `.env` files use **4 variables only**. These names are final and non-negotiable:

```
UPLOAD_PORT
MONITOR_PORT
MONITOR_SPEED
UPLOAD_SPEED
```

**There are NO board-specific port variables.** Regardless of how many boards or environments the project has, the Makefile always uses `UPLOAD_PORT` and `MONITOR_PORT`. PlatformIO's `--environment` flag selects the board — the port variable stays the same.

The following names are ALL WRONG and must NEVER appear in any Makefile or `.env` file:

- ~~ESP32_PORT~~, ~~ESP01_PORT~~, ~~ESP8266_PORT~~, ~~M5STACK_PORT~~
- ~~ESP_PORT~~, ~~DEVICE_PORT~~, ~~SERIAL_PORT~~, ~~PORT~~
- ~~ESP32_MONITOR_PORT~~, ~~ESP01_MONITOR_PORT~~, ~~BOARD_PORT~~
- ~~BAUD_RATE~~, ~~SERIAL_SPEED~~, ~~FLASH_SPEED~~
- Any `<BOARDNAME>_PORT` or `<BOARDNAME>_SPEED` pattern

These names match PlatformIO's own conventions (`upload_port`, `monitor_port`). Copy the Makefile template from **Step 6** verbatim — do not rename variables, do not add board-specific prefixes.

## Context Detection

Before starting, determine the context:

1. **Check for `platformio.ini`** in the current directory (or the target directory).

2. **If `platformio.ini` exists** → this is an existing project that needs reorganization. Follow **## Migrating an Existing PlatformIO Project** below.

3. **If `platformio.ini` does not exist** → this is a new project being created from scratch. Follow **## Creating from Scratch** below.

4. **Always follow ## Canonical Layout** — it is the target structure for both flows.

5. **Follow ## PlatformIO Configuration** when setting up or updating `platformio.ini`.

---

## Complexity Tiers

Projects fall into one of three tiers — choose based on what the user describes:

| Tier | When | Examples |
|------|------|---------|
| **Simple** | Single board, no libraries, no secrets | LED blink, basic GPIO |
| **Standard** | Single board, external libraries, secret credentials | Sensor reading with WiFi |
| **Advanced** | Multiple boards or platforms, tests, build variants | Mining firmware, proxy with web UI |

---

## Canonical Layout

Canonical structure for all PlatformIO-based Arduino projects. `platformio.ini` and `Makefile` live at the project root. Source code lives in `src/`. Build automation delegates to `make/` helper scripts.

**Key principle**: PlatformIO requires `src/` for firmware source. The Makefile wraps PlatformIO commands to provide a consistent interface. Secrets are never committed — only `.template` or `.example` files are tracked.

### Tier 1 — Simple Project

Single board, one source file, no credentials, no dependencies.

```
project-name/
├── src/
│   └── main.cpp              ← Single firmware source
├── make/
│   ├── check-pio.sh          ← Verify PlatformIO installation
│   ├── run-pio.sh            ← Execute pio in venv
│   └── clean.sh              ← Remove build artifacts
├── platformio.ini            ← Board and framework config
├── Makefile                  ← Build orchestration
├── .gitignore
├── LICENSE
└── README.md
```

### Tier 2 — Standard Project

Single board, external libraries, WiFi/API credentials.

```
project-name/
├── src/
│   ├── main.cpp              ← Main firmware
│   ├── secret.h              ← Credentials (gitignored)
│   └── secret.h.template     ← Credentials template (committed)
├── make/
│   ├── check-pio.sh
│   ├── run-pio.sh
│   ├── clean.sh
│   └── detect_board.sh       ← Auto-detect USB port
├── .vscode/
│   └── extensions.json       ← Recommend PlatformIO IDE extension
├── platformio.ini            ← Board + lib_deps
├── Makefile
├── .env.example              ← Port/speed overrides template
├── .gitignore
├── LICENSE
└── README.md
```

### Tier 3 — Advanced Project

Multiple boards or platforms (ESP32 + ESP8266), Unity tests, build variants.

```
project-name/
├── src/                      ← Firmware source (12-22 files typical)
│   ├── main.cpp
│   ├── config.h              ← Configuration constants
│   ├── module_a.cpp          ← Feature module
│   ├── module_a.h
│   └── ...
├── lib/                      ← Local libraries (when not using PlatformIO registry)
├── test/                     ← Unity unit tests
│   ├── mocks/
│   │   └── arduino_stubs.h   ← Arduino API stubs for native testing
│   └── test_<module>/
│       └── test_<module>.cpp
├── data/                     ← Static assets (SPIFFS/LittleFS filesystem image)
├── scripts/
│   ├── pio_check.sh
│   ├── build.sh              ← Multi-variant build automation
│   └── generate_assets.sh    ← Build all firmware variants
├── make/
│   ├── check-pio.sh
│   ├── run-pio.sh
│   ├── clean.sh
│   └── detect_board.sh       ← Auto-detect USB port
├── .vscode/
│   └── extensions.json
├── platformio.ini            ← Multi-environment config with inheritance
├── Makefile                  ← BOARD variable selects target environment
├── .env.example
├── .gitignore
├── LICENSE
├── README.md
└── README-PT.md
```

### Core Files

#### `src/main.cpp` — Entry Point

All PlatformIO Arduino projects use:

```cpp
#include <Arduino.h>

void setup() {
  // Runs once on boot
}

void loop() {
  // Runs repeatedly
}
```

#### `src/secret.h.template` — Credentials Template

```cpp
#pragma once

// Copy this file to secret.h and fill in your credentials.
// DO NOT commit secret.h — it is gitignored.

#define WIFI_SSID "your-ssid"
#define WIFI_PASSWORD "your-password"
```

**Important**: The variable names defined in the template (e.g., `WIFI_SSID`, `WIFI_PASSWORD`) MUST match what the source code references. If `main.cpp` uses `const char* ssid`, the template should declare `const char* ssid` — not `#define WIFI_SSID`. Choose one convention and use it consistently across both the template and all source files that include `secret.h`.

#### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "platformio.platformio-ide"
  ]
}
```

#### `.env.example`

```bash
# Copy to .env and adjust for your setup.
UPLOAD_PORT=/dev/ttyUSB0
MONITOR_PORT=/dev/ttyUSB0
MONITOR_SPEED=115200
UPLOAD_SPEED=921600
```

---

## Creating from Scratch

Step-by-step flow for new projects.

### Step 1 — Determine Tier

Ask or infer from the user's description:

| Signal | Tier |
|--------|------|
| "blink", "simple", single GPIO task | Simple |
| WiFi, sensor, API key, library needed | Standard |
| Multiple boards, tests, variants, web UI | Advanced |

### Step 2 — Choose Board Config

**Common boards:**

| Board | PlatformIO id | platform | Notes |
|-------|--------------|---------|-------|
| ESP32 DevKit | `esp32dev` | `espressif32` | Most common ESP32 |
| ESP32 M5Stack | `m5stack-core-esp32` | `espressif32` | With display |
| ESP8266 generic | `esp_wroom_02` | `espressif8266` | WiFi only, less RAM |
| ESP-01 (1MB) | `esp01_1m` | `espressif8266` | Minimal, no USB |

**Upload speed:**
- ESP32: `upload_speed = 921600`
- ESP8266: `upload_speed = 115200` (ESP-01: max 115200)

**Default ports:**
- Use `/dev/ttyUSB0` as default; document that the user may need to change it.

### Step 3 — Create Files

#### Simple tier — files to create:

1. `src/main.cpp` — minimal Arduino sketch
2. `platformio.ini` — single `[env:<board>]` section
3. `Makefile` — standard targets
4. `make/check-pio.sh`, `make/install-pio.sh`, `make/run-pio.sh`, `make/clean.sh`
5. `.gitignore` — see gitignore-skeleton for PlatformIO patterns
6. `README.md`

#### Standard tier — add to Simple:

7. `src/secret.h.template`
8. `make/detect_board.sh`
9. `.vscode/extensions.json`
10. `.env.example`
11. Add `lib_deps` to `platformio.ini`

#### Advanced tier — add to Standard:

12. `test/` structure with Unity tests
13. `data/` for filesystem assets
14. `scripts/` replacing or supplementing `make/`
15. Multi-environment `platformio.ini` (see ## PlatformIO Configuration)
16. `README-PT.md`

### Step 4 — Standard Makefile Targets

All tiers share the same Makefile targets:

```makefile
build        # pio run
upload       # pio run --target upload
flash        # build + upload
monitor      # pio device monitor
clean        # pio run --target clean
deps         # pio pkg install
check        # pio check (static analysis)
test         # pio test
erase        # pio run --target erase
detect-port  # auto-detect USB port of connected board
check-pio    # verify PlatformIO is installed
install-pio  # pip install platformio
help         # show available targets
```

For Advanced tier, add:

```makefile
BOARD ?= esp32     # selectable via: make BOARD=esp8266 flash
```

### Step 5 — make/ Scripts

#### `make/check-pio.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

if command -v pio &>/dev/null; then
    exit 0
fi

PIO_DEFAULT="$HOME/.platformio/penv/bin/pio"
if [ -f "$PIO_DEFAULT" ]; then
    exit 0
fi

echo "PlatformIO not found. Run: make install-pio"
exit 1
```

#### `make/install-pio.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

pip install platformio
```

#### `make/run-pio.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

# Add default PlatformIO install path to PATH if pio is not in PATH
PIO_DEFAULT="$HOME/.platformio/penv/bin/pio"
if [ -f "$PIO_DEFAULT" ]; then
    export PATH="$(dirname "$PIO_DEFAULT"):$PATH"
fi

exec pio "$@"
```

#### `make/clean.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"
./make/run-pio.sh run --target clean
```

#### `make/detect_board.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
ENV_FILE="$ROOT_DIR/.env"

for port in /dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyUSB2 /dev/ttyACM0; do
    if [ -e "$port" ]; then
        if [ -f "$ENV_FILE" ]; then
            sed -i "s|^UPLOAD_PORT=.*|UPLOAD_PORT=$port|" "$ENV_FILE"
            sed -i "s|^MONITOR_PORT=.*|MONITOR_PORT=$port|" "$ENV_FILE"
        else
            echo "UPLOAD_PORT=$port" > "$ENV_FILE"
            echo "MONITOR_PORT=$port" >> "$ENV_FILE"
            echo "MONITOR_SPEED=115200" >> "$ENV_FILE"
            echo "UPLOAD_SPEED=921600" >> "$ENV_FILE"
        fi
        echo "Detected board at: $port (saved to .env)"
        exit 0
    fi
done

echo "No board detected on USB ports."
echo "Checked: /dev/ttyUSB0, /dev/ttyUSB1, /dev/ttyUSB2, /dev/ttyACM0"
exit 1
```

### Step 6 — Standard Makefile Template

> **WARNING**: Copy the variable declarations below EXACTLY as written. The variable names `UPLOAD_PORT`, `MONITOR_PORT`, `MONITOR_SPEED`, and `UPLOAD_SPEED` are mandatory. Never rename them to `ESP32_PORT`, `SERIAL_PORT`, or any other name.

```makefile
MAKEFLAGS += --no-print-directory
.DEFAULT_GOAL := help

-include .env

UPLOAD_PORT      ?= /dev/ttyUSB0
MONITOR_PORT     ?= /dev/ttyUSB0
MONITOR_SPEED    ?= 115200
UPLOAD_SPEED     ?= 921600

.PHONY: build check check-pio clean deps detect-port erase flash help install-pio monitor test upload

build: check-pio ## Compile firmware
	./make/run-pio.sh run

upload: check-pio ## Upload firmware to device
	./make/run-pio.sh run --target upload \
	    --upload-port $(UPLOAD_PORT) \
	    --upload-speed $(UPLOAD_SPEED)

flash: build upload ## Compile and upload

monitor: check-pio ## Open serial monitor
	./make/run-pio.sh device monitor \
	    --port $(MONITOR_PORT) \
	    --baud $(MONITOR_SPEED)

clean: ## Remove build artifacts
	./make/clean.sh

deps: check-pio ## Install dependencies
	./make/run-pio.sh pkg install

check: check-pio ## Run static analysis
	./make/run-pio.sh check

test: check-pio ## Run unit tests
	./make/run-pio.sh test

erase: check-pio ## Erase device flash memory
	./make/run-pio.sh run --target erase

detect-port: ## Auto-detect board USB port and save to .env
	@./make/detect_board.sh

install-pio: ## Install PlatformIO
	@./make/install-pio.sh

check-pio: ## Verify PlatformIO is installed
	@./make/check-pio.sh

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) \
	    | awk 'BEGIN {FS = ":.*?## "}; {printf "  %-12s %s\n", $$1, $$2}'
```

For Advanced tier with BOARD selection, add before `.PHONY`:

```makefile
SUPPORTED_BOARDS := esp32 esp8266 esp32_oled esp8266_oled
BOARD            ?= esp32

ifeq ($(BOARD),esp32)
BUILD_ENV := esp32dev
else ifeq ($(BOARD),esp8266)
BUILD_ENV := esp_wroom_02
else ifeq ($(BOARD),esp32_oled)
BUILD_ENV := esp32dev_oled
else ifeq ($(BOARD),esp8266_oled)
BUILD_ENV := esp_wroom_02_oled
else
$(error Unsupported BOARD=$(BOARD). Supported: $(SUPPORTED_BOARDS))
endif

VERSION ?= $(shell git describe --tags --always --dirty 2>/dev/null || echo "v0.1.0-dev")
```

And update build/upload targets to use `$(BUILD_ENV)`:

```makefile
build: check-pio ## Compile firmware (BOARD=esp32|esp8266)
	./make/run-pio.sh run --environment $(BUILD_ENV)

upload: check-pio ## Upload to device (BOARD=esp32|esp8266)
	./make/run-pio.sh run --environment $(BUILD_ENV) --target upload
```

---

## Migrating an Existing PlatformIO Project

Flow for projects that already have `platformio.ini`.

> **You MUST verify and apply EVERY item in the checklist below.
> Do not skip items because the project looks "mostly correct".**

### Step 1 — Audit Current State

Read these files if they exist:
- `platformio.ini`
- `Makefile`
- `src/` directory structure
- `make/` or `scripts/` directory

Identify what is missing compared to the target layout in ## Canonical Layout.

### Step 2 — Determine Target Tier

Look at `platformio.ini` to determine current complexity:

- Single `[env:*]` section → Simple or Standard
- Multiple `[env:*]` sections → Advanced
- `platform = native` test env → Advanced with host testing

### Step 3 — Common Gaps to Fix

#### Required `make/` scripts

> **Do not create duplicate scripts.** If the project has a script like `pio_check.sh` that duplicates `check-pio.sh`, consolidate them into the standard name. The canonical names are: `check-pio.sh`, `install-pio.sh`, `run-pio.sh`, `clean.sh`, `detect_board.sh`. No other script names should exist in `make/` for these functions.

Verify each script exists. Create any that are missing using the templates in ## Creating from Scratch > Step 5:

- [ ] `make/check-pio.sh` — verifies PlatformIO is installed
- [ ] `make/install-pio.sh` — installs PlatformIO via pip
- [ ] `make/run-pio.sh` — wraps `pio` with PATH fallback
- [ ] `make/clean.sh` — delegates clean to PlatformIO

For Standard tier also check:
- [ ] `make/detect_board.sh` — auto-detects USB port

#### Required Makefile targets

Verify each target exists. Add any that are missing using the template in ## Creating from Scratch > Step 6. Do not remove project-specific targets.

| Target | Must exist in |
|--------|--------------|
| `build` | All tiers |
| `upload` | All tiers |
| `flash` | All tiers |
| `monitor` | All tiers |
| `clean` | All tiers |
| `deps` | All tiers |
| `check` | All tiers |
| `test` | All tiers |
| `erase` | All tiers |
| `detect-port` | Standard, Advanced |
| `check-pio` | All tiers |
| `install-pio` | All tiers |
| `help` | All tiers |

Also verify:
- `MAKEFLAGS += --no-print-directory` is the first line
- `.DEFAULT_GOAL := help` is the second line
- All targets are declared in a single `.PHONY` line (alphabetical)
- `check-pio` has `## Verify PlatformIO is installed` so it appears in `make help`

#### Secrets committed to git

If `secret.h` or `.env` is tracked:
1. Add to `.gitignore`
2. Create `secret.h.template` from `secret.h` (replace values with placeholders)
3. Inform the user they need to rotate any exposed credentials.

#### Missing `.env.example`

Create from the pattern in ## Canonical Layout > Core Files.

#### Missing `.vscode/extensions.json`

Create from the pattern in ## Canonical Layout > Core Files.

### Step 4 — Do Not Break

- Never change `platformio.ini` environment names without updating the Makefile.
- Never move files out of `src/` — PlatformIO requires it.
- Never remove `lib_deps` entries without checking if the code imports them.
- If `data/` exists with filesystem assets, preserve it.
- If `test/` exists, preserve all test files.

---

## PlatformIO Configuration

Reference for `platformio.ini` at each complexity tier. Always prefer the simplest tier that meets requirements.

> **NOTE on port/speed declarations**: When using the standard Makefile (which passes `--upload-port` and `--port` via CLI), the `upload_port`, `monitor_port`, `monitor_speed`, and `upload_speed` settings in `platformio.ini` serve as fallback defaults. You MAY omit them from `platformio.ini` if the Makefile provides them — but keeping them is also acceptable for IDE users who run `pio` directly. Either approach is valid; just be aware of the redundancy.

### Tier 1 — Single Board, No Libraries

```ini
[env:esp32]
platform = espressif32
board = esp32dev
framework = arduino
upload_port = /dev/ttyUSB0
monitor_port = /dev/ttyUSB0
monitor_speed = 115200
upload_speed = 921600
```

For ESP8266:

```ini
[env:esp01_1m]
platform = espressif8266
board = esp01_1m
framework = arduino
upload_port = /dev/ttyUSB0
monitor_port = /dev/ttyUSB0
monitor_speed = 115200
upload_speed = 115200
```

### Tier 2 — Single Board with Libraries

```ini
[env:esp32]
platform = espressif32
board = esp32dev
framework = arduino
upload_port = /dev/ttyUSB0
monitor_port = /dev/ttyUSB0
monitor_speed = 115200
upload_speed = 921600
lib_deps =
    adafruit/Adafruit BMP280 Library@^2.6.8
    adafruit/Adafruit Unified Sensor@^1.1.14
```

**Library declaration rules:**
- Use `owner/Library Name@^version` format from PlatformIO registry
- Pin major version with `^` for compatible updates
- Always specify version — never use unversioned deps

### Tier 3a — Multi-Board, Same Platform (ESP32 variants)

```ini
[platformio]
src_dir = src
lib_dir = lib
data_dir = data

[env]
platform = espressif32
framework = arduino
monitor_speed = 115200
lib_deps =
    bblanchon/ArduinoJson@^6.19.3

[env:esp32-release]
board = esp32dev
upload_port = /dev/ttyUSB0
upload_speed = 921600

[env:esp32-debug]
board = esp32dev
upload_port = /dev/ttyUSB0
build_type = debug

[env:m5stack-release]
board = m5stack-core-esp32
upload_port = /dev/ttyUSB0
upload_speed = 921600

[env:m5stack-debug]
board = m5stack-core-esp32
upload_port = /dev/ttyUSB0
build_type = debug
```

### Tier 3b — Multi-Platform (ESP32 + ESP8266)

```ini
[platformio]
default_envs = esp32dev

[env]
framework = arduino
monitor_speed = 115200
upload_speed = 921600
lib_deps =
    bblanchon/ArduinoJson@^6.21.3

[env:esp32dev]
platform = espressif32
board = esp32dev
upload_port = /dev/ttyUSB0
monitor_port = /dev/ttyUSB0
lib_deps =
    ${env.lib_deps}
    me-no-dev/AsyncTCP@^1.1.1

[env:esp_wroom_02]
platform = espressif8266
board = esp_wroom_02
upload_port = /dev/ttyUSB0
upload_speed = 115200
board_build.filesystem = littlefs
lib_deps =
    ${env.lib_deps}
    me-no-dev/ESPAsyncTCP@^1.2.2

[env:esp32dev_oled]
extends = env:esp32dev
build_flags =
    ${env:esp32dev.build_flags}
    -DUSE_OLED_STATUS

[env:esp_wroom_02_oled]
extends = env:esp_wroom_02
build_flags =
    ${env:esp_wroom_02.build_flags}
    -DUSE_OLED_STATUS
```

**Key patterns:**
- `${env.lib_deps}` inherits from the base `[env]` section
- `extends = env:base` copies all settings from another env
- Build variants (OLED/standard) use `extends` + additional `build_flags`
- `-DFLAG_NAME` in `build_flags` defines a compile-time constant

### Tier 3c — With Unity Tests

Add to any multi-board config:

```ini
[env:test]
platform = espressif32
board = esp32dev
test_framework = unity
test_build_src = yes
test_ignore = **/main.cpp

[env:native-<module>]
platform = native
test_framework = unity
test_build_src = yes
test_filter = test_<module>
build_flags =
    -DUNIT_TEST
    -Isrc
    -Itest/mocks
build_src_filter = +<<module>.cpp>
```

**Test directory structure:**

```
test/
├── mocks/
│   └── arduino_stubs.h    ← Stub Arduino.h, Serial, etc. for native builds
└── test_<module>/
    └── test_<module>.cpp  ← Unity test file
```

**`test/mocks/arduino_stubs.h` minimum content:**

```cpp
#pragma once
#ifdef UNIT_TEST

#include <stdint.h>
#include <stdio.h>
#include <string.h>

// Minimal Arduino stubs for native unit testing
#define HIGH 1
#define LOW  0
#define INPUT 0
#define OUTPUT 1

void pinMode(uint8_t, uint8_t) {}
void digitalWrite(uint8_t, uint8_t) {}
int  digitalRead(uint8_t) { return 0; }
void delay(unsigned long) {}
unsigned long millis() { return 0; }

class HardwareSerial {
public:
    void begin(long) {}
    void println(const char* s) { printf("%s\n", s); }
    void print(const char* s) { printf("%s", s); }
};
extern HardwareSerial Serial;
HardwareSerial Serial;

#endif // UNIT_TEST
```

### Common `build_flags`

| Flag | Purpose |
|------|---------|
| `-DCORE_DEBUG_LEVEL=3` | Enable verbose Arduino debug output |
| `-DUSE_OLED_STATUS` | Enable OLED display feature |
| `-DUNIT_TEST` | Guard stubs for native test builds |
| `-DFIRMWARE_VERSION='"$(VERSION)"'` | Embed version string from Makefile |

### Filesystem Support

| Platform | Filesystem | Setting |
|----------|-----------|---------|
| ESP32 | SPIFFS (legacy) | `board_build.filesystem = spiffs` |
| ESP32 | LittleFS (preferred) | `board_build.filesystem = littlefs` |
| ESP8266 | LittleFS | `board_build.filesystem = littlefs` |

When using filesystem:
- Put static files in `data/` directory
- Add `upload_filesystem` target to Makefile:

```makefile
upload-fs: check-pio ## Upload filesystem image
	./make/run-pio.sh run --target uploadfs
```

---

## Chaining

When creating a complete project from scratch, the full workflow involves these skills in order:

1. **`esp32-skeleton`** — create the PlatformIO structure and Makefile
2. **`gitignore-skeleton`** — `.gitignore` with PlatformIO artifacts, secrets, and editor dirs
3. **`readme-skeleton`** — `README.md` with the standard structure

After completing a migration:

1. **Check the Makefile** — verify it matches the standard in ## Creating from Scratch > Step 6
2. **Check the `.gitignore`** — if it is missing the AI Tools section or deviates from the standard, invoke the `gitignore-skeleton` skill
3. **Check the READMEs** — if `README.md` or `README-PT.md` need updating, invoke the `readme-skeleton` skill

## Related Skills

- **gitignore-skeleton** — Standard `.gitignore` adapted for PlatformIO projects
- **readme-skeleton** — Standard README content and bilingual conventions
- **makefile-skeleton** — Makefile patterns (the ESP32 Makefile follows similar conventions)
