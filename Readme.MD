# ESP32 AI-on-the-Edge (Build-only with Memfault integration)

## Overview
This repository contains a fork/integration of the "AI-on-the-edge-device" ESP32-CAM project with Memfault SDK integration. It focuses on achieving a clean, reproducible build and wiring up metric definitions. It does not aim to run in an emulator nor to upload data to Memfault without hardware.

## Current Status
- **Builds successfully** with PlatformIO for `env: esp32cam`.
- **Memfault build ID** is generated and injected post-link.
- **Metrics defined** in `code/main/memfault/memfault_metrics_heartbeat_config.def`:
  - `cpu_temp`
  - `wifi_rssi`
  - `recognition_time_seconds`
- Minor fixes applied:
  - Corrected Memfault metrics `.def` syntax (removed trailing `;`).
  - Fixed RSSI call site in `code/components/jomjol_flowcontroll/MainFlowControl.cpp` to `get_WIFI_RSSI()`.
  - Corrected post-link script path in `code/add_build_id.py`.

## What this project does
- Builds the firmware and produces artifacts:
  - `code/.pio/build/esp32cam/firmware.elf`
  - `code/.pio/build/esp32cam/firmware.bin`
- Integrates Memfault SDK headers and post-link build ID step.
- Demonstrates where and how metrics are set during the flow task.

## What this project does NOT do
- Does **not** emulate ESP32 runtime (no Renode/QEMU support for this stack).
- Does **not** send Memfault chunks to the cloud without running on real hardware.
- Does **not** verify Wi‑Fi, camera, SD, or HTTP server behaviour in CI.

## Prerequisites
- PlatformIO CLI installed.
- macOS environment - Apple Silicon (tested) with ESP32 toolchains via PlatformIO.

## Build
From the `code/` directory:

```bash
pio run -e esp32cam
```

Artifacts will be in `code/.pio/build/esp32cam/`.

## Upload (optional, requires real hardware)
```bash
pio run -e esp32cam -t upload
```
If PlatformIO does not auto-detect the port, pass `--upload-port /dev/tty.usbserial-XXXX`.

## Where metrics are defined and used
- Definitions: `code/main/memfault/memfault_metrics_heartbeat_config.def`
- Usage (examples): `code/components/jomjol_flowcontroll/MainFlowControl.cpp`
  - `MEMFAULT_METRIC_SET_SIGNED(recognition_time_seconds, ...)`
  - `MEMFAULT_METRIC_SET_SIGNED(cpu_temp, temperatureRead())`
  - `MEMFAULT_METRIC_SET_SIGNED(wifi_rssi, get_WIFI_RSSI())`

## Notes on emulation
There is no practical emulator for this project’s Xtensa + camera + Wi‑Fi + FS + HTTP stack. Build-only validation works; runtime requires an ESP32(-CAM) board.

## Repository layout highlights
- `AI-on-the-edge-device/code/` — PlatformIO project root
  - `components/` — project and SDK components
  - `main/` — app entry and configuration
  - `.pio/` — build outputs (generated)

## Troubleshooting
- Memfault build ID script path is configured in `code/add_build_id.py`.
- If build fails with missing script, ensure SDK path is `components/memfault-firmware-sdk/`.
- If upload fails: check serial port and board in bootloader mode.

## Resources
Followed this video https://www.youtube.com/watch?v=yBrUJVTGdRM
Documentation (it is outdated, be careful with their code snippets!) https://docs.memfault.com/docs/mcu/platformio-esp-idf-guide 
