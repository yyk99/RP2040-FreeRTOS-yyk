# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

This project uses CMake targeting RP2040 (Cortex-M0+) with the Pico SDK. There are no host-side tests; all output is firmware (`.uf2` files).

### Configure (using external Pico SDK at `/opt/pico-sdk`)

```shell
# For Pico 2 W
cmake -S . -B build/pico2_w -DPICO_BOARD=pico2_w -DPICO_SDK_PATH=/opt/pico-sdk

# For original Pico
cmake -S . -B build/pico -DPICO_SDK_PATH=/opt/pico-sdk
```

### Build

```shell
cmake --build build/pico2_w
# or
cmake --build build/pico
```

### Debug vs Release

```shell
cmake -S . -B build/pico -DPICO_SDK_PATH=/opt/pico-sdk -DCMAKE_BUILD_TYPE=Debug
cmake -S . -B build/pico -DPICO_SDK_PATH=/opt/pico-sdk -DCMAKE_BUILD_TYPE=Release
```

`DEBUG=1` is automatically set for Debug builds (via CMake generator expression), which enables UART output via `log_debug()` and USB stdio initialization.

### Deploy firmware to device

```shell
./deploy.sh /dev/ttyACM0 build/pico/App-Template/TEMPLATE.uf2
./deploy.sh /dev/ttyACM0 build/pico/App-Scheduling/SCHEDULING_DEMO.uf2
```

## Architecture

### FreeRTOS Integration

FreeRTOS is built as a static library (`FreeRTOS`) in the top-level `CMakeLists.txt` using the `GCC/ARM_CM0` portable port (Cortex-M0+ for RP2040) and `heap_3` memory management. Every app links against `pico_stdlib` and `FreeRTOS`. FreeRTOS configuration is in `Config/FreeRTOSConfig.h`; ISR handlers are remapped to Pico SDK names there (`isr_svcall`, `isr_pendsv`, `isr_systick`).

### App Structure

All four apps follow the same pattern: create FreeRTOS tasks in `main()`, set up an `xQueueCreate` for inter-task communication, then call `vTaskStartScheduler()`. The scheduler never returns.

- **App-Template** (`main.c`, C): Two tasks communicate LED state via `xQueue`. Pico W boards use the CYW43 WiFi chip GPIO for the LED; standard Pico uses `PICO_DEFAULT_LED_PIN`. Both are handled with `#if defined(PICO_DEFAULT_LED_PIN) / #elif defined(CYW43_WL_GPIO_LED_PIN)` guards.
- **App-Scheduling** (`main.cpp`, C++): Adds MCP9808 temperature sensor and HT16K33 LED display.
- **App-IRQs** (`main.cpp`, C++): Builds on App-Scheduling; uses MCP9808 temperature alert pin to trigger an interrupt/semaphore.
- **App-Timers** (`main.cpp`, C++): Demonstrates FreeRTOS software timers; no extra hardware required.

### Common Library

`Common/` holds C++ driver code shared by Apps 2-4:
- `ht16k33.cpp/h` - HT16K33 4-digit 7-segment display driver (I2C)
- `mcp9808.cpp/h` - MCP9808 temperature sensor driver (I2C)
- `i2c_utils.cpp/h` - I2C bus helpers
- `utils.cpp/h` - Shared utilities

### Submodules

- `FreeRTOS-Kernel/` - FreeRTOS kernel (do not recurse when updating)
- `pico-sdk` - May be replaced by an external SDK via `-DPICO_SDK_PATH`

To initialize submodules after cloning:
```shell
git submodule update --init   # Do NOT use --recursive
cd pico-sdk && git submodule update --init && cd ..
```

### Excluding Apps from Build

Comment out `add_subdirectory()` lines in the top-level `CMakeLists.txt` to exclude specific apps from the build.
