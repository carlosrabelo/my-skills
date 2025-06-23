# PlatformIO Configuration Patterns

Reference for `platformio.ini` at each complexity tier. Always prefer the simplest tier that meets requirements.

---

## Tier 1 — Single Board, No Libraries

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

---

## Tier 2 — Single Board with Libraries

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

---

## Tier 3a — Multi-Board, Same Platform (ESP32 variants)

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

---

## Tier 3b — Multi-Platform (ESP32 + ESP8266)

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

---

## Tier 3c — With Unity Tests

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

---

## Common `build_flags`

| Flag | Purpose |
|------|---------|
| `-DCORE_DEBUG_LEVEL=3` | Enable verbose Arduino debug output |
| `-DUSE_OLED_STATUS` | Enable OLED display feature |
| `-DUNIT_TEST` | Guard stubs for native test builds |
| `-DFIRMWARE_VERSION='"$(VERSION)"'` | Embed version string from Makefile |

---

## Filesystem Support

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
