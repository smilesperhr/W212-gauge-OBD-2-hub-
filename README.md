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
| Code library | 286 codes for the M272 engine and 722.9 transmission, each with a severity score and likely fix. Live data narrows the cause, e.g. a single-cylinder misfire with normal fuel trims points to the coil pack; a whole bank running lean points to fuel or a vacuum leak. |
| Hub website | Join the hub's Wi-Fi from a phone: live data, CAN sniffer, trip stats, log downloads, code details, clear codes and firmware updates. The diag screen shows a QR code that opens a code's details. |

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

## Current features

All of these are built and running in the car.

**Footwell module**
- Reads the car over CAN and finds which readings the ECU supports
- Sends live data to both screens 25 times a second over ESP-NOW, and takes commands back from the 2.4
- Reads stored and pending codes, misfire counts per cylinder and the rear O2 sensors
- Clears codes (engine off only), saving a record of them first
- Logs every trip once a second, plus a summary at shut-off
- Saves a freeze-frame when a new code appears: the last 30 seconds of data plus raw CAN
- Over-the-air firmware updates, and sleeps 5 minutes after the car is off

**Website (W212-HUB Wi-Fi)**
- Live: every value the car reports, codes and clear codes
- Sniffer: every CAN ID live, snapshots and 60-second raw recordings
- Trip: distance, fuel, average MPG, max speed, RPM and coolant, 0-60
- Dump: all trips, CAN logs and freeze-frames in one zip
- Full details for any of the 286 codes, and firmware upload

**2.4 in diag screen**
- Eight pages: Temps, Performance, Fuel trims, Misfires, Engine, Trip, Codes, Clear codes
- 0-60, 30-70, quarter mile and trap speed, with best times saved
- Diag pop-up with each code's meaning, severity and likely fix; misfires show the cylinder map and whether the fix is spark-side or air-side
- QR code that opens a code's details on a phone

**2.8 in gauge**
- "Welcome, Mr. Parsons" startup with a chime, then the gauge fades in
- Shift point at 6,200 rpm: exhaust flames, flashing hazards, strobing border and shift-light bar
- Cold start: snow and frost melt, then a checkered flag and "Safe to engage Sport mode"
- Check engine badge, with the sky color following the code's severity

## Status

The footwell module, both screens and the website are running in the car. Now fine-tuning and adding features. Coming next: a radar detector module, weather alerts, a custom voice greeting and cloud trip logging.
