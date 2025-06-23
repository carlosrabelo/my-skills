# ESP32/ESP8266 — Standard Layout

Canonical structure for all PlatformIO-based Arduino projects. `platformio.ini` and `Makefile` live at the project root. Source code lives in `src/`. Build automation delegates to `make/` helper scripts.

**Key principle**: PlatformIO requires `src/` for firmware source. The Makefile wraps PlatformIO commands to provide a consistent interface. Secrets are never committed — only `.template` or `.example` files are tracked.

---

## Tier 1 — Simple Project

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

---

## Tier 2 — Standard Project

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

---

## Tier 3 — Advanced Project

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
│   ├── detect_board.sh
│   ├── build.sh              ← Multi-variant build automation
│   └── generate_assets.sh    ← Build all firmware variants
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

---

## Core Files

### `src/main.cpp` — Entry Point

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

### `src/secret.h.template` — Credentials Template

```cpp
#pragma once

// Copy this file to secret.h and fill in your credentials.
// DO NOT commit secret.h — it is gitignored.

#define WIFI_SSID "your-ssid"
#define WIFI_PASSWORD "your-password"
```

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "platformio.platformio-ide"
  ]
}
```

### `.env.example`

```bash
# Copy to .env and adjust for your setup.
UPLOAD_PORT=/dev/ttyUSB0
MONITOR_PORT=/dev/ttyUSB0
MONITOR_SPEED=115200
UPLOAD_SPEED=921600
```
