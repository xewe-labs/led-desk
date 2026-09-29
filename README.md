# XeWe LED - Desk — ESP8266 desk and ceiling lights for HomeKit and Alexa

Personal project · 2022-10 → 2024-07 · Solo: Max Dokukin · Status: Completed (Release V1.0 2023-11-12, last change 2024-07-01)

![Desk lit blue by the LED strip](static/media/resources/IMG_0654.webp)

## Overview

Smarthome Lights ESP is an ESP8266 sketch that connects addressable LED strips to the Apple HomeKit and Amazon Alexa
smart-home systems. One board drives two strips — a 61-LED strip behind the desk and a 276-LED strip on the ceiling —
and exposes each one to HomeKit as a colour lightbulb plus a "Fade" switch, and to Alexa as a colour light plus an
on/off "Fade" device. Each strip has two modes: solid colour, and Perlin fade (pretty waves) — animated noise around the
chosen hue. Colour, brightness and hue changes fade over 900 ms, and the last colour, brightness, mode and power state
are saved to EEPROM so the lights come back the way they were after a power cut.

## Highlights

- Dual ecosystem control from one board: HomeKit (Arduino-HomeKit-ESP8266, bridge + 4 accessories) and Alexa (Espalexa, 4 devices) — `my_accessory.c`, `Alexa.h`
- Two strips, 337 LEDs in total (61 desk on `D1`, 276 ceiling on `D2`) driven by one reusable `LedController` class — `Smarthome-Lights-ESP-Desk.ino`
- Perlin-noise "fade" mode built on FastLED `inoise8`, with a narrower hue spread for reds — `PerlinFade.h`
- Smooth 900 ms colour, brightness and Perlin-hue transitions — `LedController.h`, `PerlinFade.h`
- State (RGB, brightness, mode, on/off) persisted in 6 EEPROM bytes per strip — `MemoryController.h`

## How it works

```
Home app ─────────HomeKit──┐                          ┌→ LedController(desk,    61 LEDs, EEPROM @1500) → NeoPixel D1
                           ├→ setters → LedController ┤
Alexa app / voice ─Espalexa┘      ↑                   └→ LedController(ceiling, 276 LEDs, EEPROM @1506) → NeoPixel D2
                                  └── valuesUpdated → syncValuesHomekit() + syncValuesAlexa()
```

`loop()` runs the HomeKit server and Espalexa, renders one frame per strip, and — whenever a setter changed something —
pushes the new state back into both ecosystems so the Home app and the Alexa app agree.

- **`Smarthome-Lights-ESP-Desk.ino`** — pins and LED counts, Wi-Fi connect, EEPROM (4096 bytes), controller construction, main loop (`delay(10)`).
- **`LedController.h`** — per-strip state machine (`SOLID_COLOR`, `SOLID_COLOR_TRANSITION`, `PERLIN`; `ON`/`OFF`), 10 ms frame gate, linear colour and brightness fades, HSV ↔ RGB bookkeeping, EEPROM writes. Perlin mode switches itself off for near-white requests (saturation below 35).
- **`PerlinFade.h`** — noise animation: each LED's colour comes from `inoise8(i * 10, t)`, mapped to a hue window (10000 of 65535, ×0.2 near red), saturation 245–255 and brightness 100–255; hue changes glide over 900 ms.
- **`Color.h`** — colour mixing, RGB → HSV and HSV → RGB conversion.
- **`HomeKit.h` + `my_accessory.c`** — HomeKit bridge "XeWe Lights" with two lightbulb accessories (on, brightness, saturation, hue) and two switches that toggle Perlin mode.
- **`Alexa.h`** — Espalexa devices "Desk Lights", "Desk Lights Fade", "Ceiling Lights", "Ceiling Lights Fade".
- **`MemoryController.h`** — reads/writes the 6-byte state block (R, G, B, brightness, mode, on/off) at a per-strip offset.
- **`Archived Versions/`** — earlier generations kept for reference: Alexa-only sketch, HomeKit V1, V2, V3 (EEPROM memory, with ESP crash logs).

## Results

| Metric | Value | Baseline / note |
|---|---|---|
| Strips / LEDs driven | 2 strips, 337 LEDs (61 desk + 276 ceiling) | Ceiling strip was 151 LEDs until 2024-06-15 |
| Smart-home endpoints | HomeKit: bridge + 2 lightbulbs + 2 switches; Alexa: 4 devices | `my_accessory.c`, `Alexa.h` |
| Transition time | 900 ms (colour, brightness, Perlin hue) | `LedController.h`, `PerlinFade.h` |
| Firmware size | 1,038 lines in 9 source files | `wc -l`, excluding `wifi_info.h` |
| History | 40 commits, 2023-10-13 → 2026-04-22; Release V1.0 on 2023-11-12 | git log |

This is a working home device rather than a measured system; the table lists what the firmware supports.

## Getting started

```text
1. Arduino IDE with the ESP8266 board package; select your ESP8266 board (pins D1/D2).
2. Install libraries: Arduino-HomeKit-ESP8266, Espalexa, Adafruit NeoPixel, FastLED.
3. Provide wifi_info.h with a wifi_connect() function for your network (keep it out of version control).
4. Set DESK_LED_NUM / CEILING_LED_NUM and the pins in Smarthome-Lights-ESP-Desk.ino, then upload.
5. Pair "XeWe Lights" in the Home app; ask Alexa to discover devices.
```

Serial monitor at 115200 baud shows the controller debug output. Known issue in the committed code: the three
`setMode(SOLID_COLOR)` lines added on 2024-07-01 in `LedController.h` are missing a semicolon — add them before
compiling. Open item from `todo.md`: a memory leak in the HomeKit library's `write(client_context_t *context, …)`.

## Documents

- [To-do list](todo.md)
- [Archived firmware versions and ESP crash logs](Archived%20Versions/)
- Photos: [desk, blue](static/media/resources/IMG_0654.webp) · [desk, red](static/media/resources/IMG_0559.webp) · [desk, purple](static/media/resources/IMG_0655.webp)
- Original project photo: [IMG_1931](https://github.com/xeweva/Smarthome-Lights-ESP/assets/54597813/85e34b70-14e2-4198-ad93-fff34ed6da69)
- Related: [XeWe LED OS](https://github.com/xewe-labs/xewe-led-os) — modular ESP32 firmware for addressable LED strips ([project page](https://maxdokukin.com/projects/xewe-led-os))
- Project page: [maxdokukin.com/projects/xewe-led-desk](https://maxdokukin.com/projects/xewe-led-desk)
