# Creating a New ESP32/ESP8266 Project

Step-by-step flow for new projects. Always read `layout.md` first.

---

## Step 1 — Determine Tier

Ask or infer from the user's description:

| Signal | Tier |
|--------|------|
| "blink", "simple", single GPIO task | Simple |
| WiFi, sensor, API key, library needed | Standard |
| Multiple boards, tests, variants, web UI | Advanced |

---

## Step 2 — Choose Board Config

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

---

## Step 3 — Create Files

### Simple tier — files to create:

1. `src/main.cpp` — minimal Arduino sketch
2. `platformio.ini` — single `[env:<board>]` section
3. `Makefile` — standard targets
4. `make/check-pio.sh`, `make/install-pio.sh`, `make/run-pio.sh`, `make/clean.sh`
5. `.gitignore` — see gitignore-skeleton for PlatformIO patterns
6. `README.md`

### Standard tier — add to Simple:

7. `src/secret.h.template`
8. `make/detect_board.sh`
9. `.vscode/extensions.json`
10. `.env.example`
11. Add `lib_deps` to `platformio.ini`

### Advanced tier — add to Standard:

12. `test/` structure with Unity tests
13. `data/` for filesystem assets
14. `scripts/` replacing or supplementing `make/`
15. Multi-environment `platformio.ini` (see `platformio.md`)
16. `README-PT.md`

---

## Step 4 — Standard Makefile Targets

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
check-pio    # verify PlatformIO is installed
install-pio  # pip install platformio
help         # show available targets
```

For Advanced tier, add:

```makefile
BOARD ?= esp32     # selectable via: make BOARD=esp8266 flash
```

---

## Step 5 — make/ Scripts

### `make/check-pio.sh`

```bash
#!/bin/bash
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

### `make/install-pio.sh`

```bash
#!/bin/bash
set -euo pipefail

pip install platformio
```

### `make/run-pio.sh`

```bash
#!/bin/bash
set -euo pipefail

# Add default PlatformIO install path to PATH if pio is not in PATH
PIO_DEFAULT="$HOME/.platformio/penv/bin/pio"
if [ -f "$PIO_DEFAULT" ]; then
    export PATH="$(dirname "$PIO_DEFAULT"):$PATH"
fi

exec pio "$@"
```

### `make/clean.sh`

```bash
#!/bin/bash
set -euo pipefail
ROOT_DIR="$(cd "$(dirname "$0")/.." && pwd)"
cd "$ROOT_DIR"
./make/run-pio.sh run --target clean
```

### `make/detect_board.sh`

```bash
#!/bin/bash
set -euo pipefail

# Detect connected ESP32/ESP8266 board USB port
for port in /dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyUSB2 /dev/ttyACM0; do
    if [ -e "$port" ]; then
        echo "Detected board at: $port"
        echo "UPLOAD_PORT=$port"
        echo "MONITOR_PORT=$port"
        exit 0
    fi
done

echo "No board detected on USB ports."
echo "Checked: /dev/ttyUSB0, /dev/ttyUSB1, /dev/ttyUSB2, /dev/ttyACM0"
exit 1
```

---

## Step 6 — Standard Makefile Template

```makefile
MAKEFLAGS += --no-print-directory
.DEFAULT_GOAL := help

-include .env

UPLOAD_PORT      ?= /dev/ttyUSB0
MONITOR_PORT     ?= /dev/ttyUSB0
MONITOR_SPEED    ?= 115200
UPLOAD_SPEED     ?= 921600

.PHONY: build check check-pio clean deps erase flash help install-pio monitor test upload

build: check-pio ## Compile firmware
	./make/run-pio.sh run

upload: check-pio ## Upload firmware to device
	./make/run-pio.sh run --target upload \
	    --upload-port $(UPLOAD_PORT)

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
