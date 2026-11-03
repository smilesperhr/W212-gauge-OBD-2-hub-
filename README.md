# W212 Gauge · OBD2 Hub

A custom ESP32 gauge and diagnostic display for a 2011 Mercedes E350 (W212, M272 V6, 7G-Tronic).

**Live pages:** https://smilesperhr.github.io/W212-gauge-OBD-2-hub-/

## What it does

A small ESP32 "footwell module" plugs into the car's OBD2 port and reads live data over CAN. It sends that data wirelessly to two screens mounted in the dash:

- **2.8 in gauge screen:** a pixel-art night drive with my own W212 on the road. Speed, RPM and a shift light where the hazards and streetlights flash and the exhaust throws flames at the shift point. Snow melts off the car as the engine warms up, then a checkered flag shows "Safe to engage Sport mode".
- **2.4 in diag screen:** temps, fuel trims, misfires, engine data, trip, stored and pending codes, and a clear-codes page. When a check engine code appears, a diag pop-up shows the code, a severity score out of 10, the M272 cylinder layout for misfires, and the most likely fix based on live data.

## How it works

| Part | Job |
|---|---|
| Footwell module (ESP32 + SN65HVD230 CAN transceiver) | Sends OBD2 requests on the 500 kbps CAN bus through the W212 gateway, asking for the next reading as soon as the last reply arrives so RPM and speed stay fast and smooth. Reads RPM, speed, temperatures, fuel trims and codes, and checks each code against a built-in library. |
| Power | OBD2 pin 16 (12V) through a 12V-to-5V converter. Pins 4/5 ground. |
| Link between boards | ESP-NOW, both ways: the hub broadcasts live data, and each screen can send commands back (open diag mode, clear codes). |
| Code library | The most common M272 / 722.9 codes with severity scores. Live data narrows the cause, e.g. a single-cylinder misfire with normal fuel trims points to the coil pack; a whole bank running lean points to fuel or a vacuum leak. |
| Hub website | Join the hub's Wi-Fi from a phone for full code details and clear codes. The diag screen shows a QR code that does this. |

## Hardware

- ELEGOO ESP32-WROOM-32 (footwell module)
- MELIFE 2.8 in ESP32-32E "CYD" display (ILI9341, resistive touch)
- DORAGA 2.4 in ESP32-2432S024 display (ILI9341, resistive touch)
- SN65HVD230 CAN transceiver, 12V-to-5V converter, PAM8406 amp and speaker

Screens run on LovyanGFX, with the screen settings kept in the sketch so library updates can't break the display.

## Pages in this repo

| Page | What it is |
|---|---|
| [W212 Gauge Test Bench](gauge-test-bench.html) | The finished pixel-art gauge, interactive. The reference the firmware matches. |
| [Vent Screen Test Bench](vent-screen-test-bench.html) | Simulates the ESP32 screens at real size: SPI speed, redraw methods, touch lag. |
| [2.4 in Diag Screen](diag-screen.html) | Diagnostic page designs. |
| [Screen Mounts](screen-mounts.html) | Two to-scale mounting options. |
| [Vent Gauge Faces](gauge-faces.html) | Early dial styles. |
| [Firmware v0.1](firmware-v0-1.html) · [v0.2](firmware-v0-2.html) | First CAN firmware notes and code. |

## Status

The screens and firmware run on the bench. The footwell module is being built next, followed by a radar detector module and a phone companion app.
