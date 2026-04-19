

https://github.com/user-attachments/assets/7ee4066d-8793-4969-9541-1f2eb0662f15

# PoliceFlash

Two-channel police-style LED flasher for the **ESP32-S3-DevKitC-1**, built with PlatformIO + Arduino framework. Runs several blink patterns and lets you switch between them live over the serial console.

## Hardware

- **MCU:** ESP32-S3 @ 240 MHz (variant `N16R8` — 16 MB Flash, 8 MB PSRAM)
- **Board:** `esp32-s3-devkitc-1`
- **LEDs:**
  - Red → **GPIO 4**
  - Blue → **GPIO 5**

Wire each LED through a current-limiting resistor to ground.

## Runtime Control

After flashing, open the serial monitor at **115200 baud**. Type a command and press Enter:

| Command | Pattern                              |
|---------|--------------------------------------|
| `p0`    | Alternating — red/blue ping-pong     |
| `p1`    | DoubleBlinkPolice *(default)*        |
| `p2`    | SOS — Morse `... --- ...` on both    |
| `?`     | Print current pattern name           |

## Architecture

The firmware uses a small strategy + builder layout so patterns and output drivers can be swapped independently.

```
main.cpp
  │
  ├── Flasher  ──  ticks through the active IPattern and writes masks to the output
  │     │
  │     ├── IOutputStrategy  ←  MultiPinOutput (digitalWrite on N pins)
  │     └── IPattern         ←  AlternatingPattern
  │                             DoubleBlinkPolicePattern
  │                             SosPattern
  │
  └── FlasherBuilder  ──  fluent construction: .withOutput(...).withPattern(...).build()
```

### Adding a new pattern

1. Create `include/patterns/MyPattern.h` deriving from `pflash::IPattern`.
2. Create `src/patterns/MyPattern.cpp` with a `constexpr Frame kFrames[]` table — each frame is `{ duration_ms, channel_mask }`.
3. Instantiate it in `main.cpp` and hook it up to a serial command.

A `Frame`'s `channel_mask` bit `N` maps to the LED at index `N` in the `MultiPinOutput` pin list.

## Project Layout

```
include/          project headers
├── Flasher.h, FlasherBuilder.h
├── IOutputStrategy.h, IPattern.h
├── outputs/      concrete output strategies
└── patterns/     concrete pattern headers
src/              matching .cpp files + main.cpp
lib/              local libraries
test/             unit tests (PlatformIO)
platformio.ini    build environment
```
