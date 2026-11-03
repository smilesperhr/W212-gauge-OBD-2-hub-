# W212 CAN Reference

Where the car's data lives and how to reach it. Every item names the document it came from.

## Why the OBD port isn't enough

The OBD port reaches the diagnostic CAN through the gateway, so the hub only sees answers to its own requests (engine computer at 7E8, transmission computer at 7E9). The modules' constant broadcast traffic (gear, E/S program, wheel speeds, steering) stays on the internal networks below. To read it, the hub listens directly on one of them, in listen-only mode so it can never transmit or disturb the car.

## The W212's CAN distributors

From Mercedes WIS document **GF00.19-P-0800FL, "CAN electrical connector, as-built configuration", Model 212.0** [1]. Each network meets at a "potential distributor", a junction where every module on that network connects.

| Distributor | Network | Location (from the WIS top view and its listed ground points) | Modules relevant to this project |
|---|---|---|---|
| **X30/30** | Vehicle floor chassis CAN | Left (driver's) side, near the left footwell ground point W15/2 | Instrument cluster (A1), ME-SFI engine computer, electronic transmission control, ESP, front SAM, electronic ignition lock (N73), steering column module (N80), tire pressure monitor (N88), airbag (N2/10) |
| **X30/21** | Drive train CAN | Right (passenger) side, near the right footwell ground point W15/1 | ME-SFI engine computer (N3/10), fully integrated transmission control unit (Y3/8n4, 722.9), DIRECT SELECT intelligent servo module (A80), front SAM, fuel pump control (N118) |
| X30/28 | Vehicle dynamics CAN | Right side | DISTRONIC PLUS (A89), yaw rate sensor (B24/15), radar sensors (N62/1) |
| X30/32 | Left vehicle floor interior CAN | Left side | Instrument cluster, front SAM, climate control (N22/7), overhead control panel (N70), door and seat modules |
| X30/33 | Right vehicle floor interior CAN | Right side | Radio / COMAND, rear SAM (N10/2), right door and seat modules |
| X30/35 | Telematics CAN | Right side | COMAND display and controls, TV tuner |

Wiring diagrams referenced by the WIS page: drive train CAN PE00.19-P-2170-99DAA, vehicle dynamics CAN PE00.19-P-2190-99DAA, chassis CAN PE00.19-P-2195-99DAA, interior CAN PE00.19-P-2185-99DAA / -99DAB, telematics CAN PE00.19-P-2180-99DAA.

### Best tap for gear and E/S

**Chassis CAN at X30/30, in the driver's footwell.** The transmission controller, engine computer, ESP and the instrument cluster all share it, and the cluster displays the gear and E/S program, so that data has to travel on this network to reach it. It is also the closest distributor to the footwell module. The drive train CAN at X30/21 (passenger footwell) is the backup: the transmission and the DIRECT SELECT shifter module live there.

Before tapping: confirm the distributor's location and a free or back-probe-able position with the chassis CAN wiring diagram (PE00.19-P-2195-99DAA). Listen-only mode; no cutting.

### Ambient lighting

The overhead control panel (N70) and front SAM sit on the **interior CAN**, not the chassis or drive train CAN. Controlling the factory lights would mean transmitting on the interior network, which this project does not do.

## Known W212 signals (Racelogic)

From the Racelogic VBOX CAN file **"Mercedes E-Class W212 2010-2016"** [2]. These are signals Racelogic decoded for the W212; the file does not say which network they are on. Once the hub is tapped in, finding these IDs confirms it's on the right network. Gear and E/S are **not** in this file and still need to be decoded.

| Signal | CAN ID | Start bit | Length | Scale | Offset | Unit | Byte order |
|---|---|---|---|---|---|---|---|
| Engine speed | 0x105 (261) | 48 | 16 | 1 | 0 | rpm | Motorola |
| Throttle position | 0x105 (261) | 24 | 8 | 1 | 0 | % (max 250) | Motorola |
| Wheel speed FL | 0x203 (515) | 48 | 14 | 0.061 | 0 | km/h | Motorola |
| Wheel speed FR | 0x203 (515) | 32 | 14 | 0.061 | 0 | km/h | Motorola |
| Wheel speed RL | 0x203 (515) | 16 | 14 | 0.061 | 0 | km/h | Motorola |
| Wheel speed RR | 0x203 (515) | 0 | 14 | 0.061 | 0 | km/h | Motorola |
| Steering angle | 0x003 (3) | 48 | 16 | 0.5 | -2050 | ° (±720) | Motorola |
| Brake position | 0x005 (5) | 32 | 12 | 0.0244141 | 0 | % | Motorola |

(The file also lists each wheel speed in mph, scale 0.037881.) Racelogic's start-bit convention for Motorola signals should be confirmed against live data before relying on it.

What these add over OBD: four wheel speeds (wheelspin and launch detection, accurate 0-60), steering angle, brake position, and RPM at the car's broadcast rate instead of request-reply.

## Broadcast traffic at the OBD port (2011 E350, confirmed)

Raw captures from the hub (files 0032 and 0033, Nov 3, 2026, engine idling) show the car's broadcast frames reaching the OBD port, not just diagnostic replies. IDs seen: 001, 003, 005, 007, 045, 04B, 105, 10C, 12D, 19F, 1C1, 201, 203, 204, 205, 206, 207, 208, 245, 26F, 283, 2EF, 319, 375, 39D, 39F, 3B7, 3E3, 3E5, 3E7, 3E9, 3EB, 3F6, 400, 413.

| Signal | ID | Decoding | Rate | Check |
|---|---|---|---|---|
| Engine speed | 0x105 | Bytes 1–2, big-endian, 1 rpm per bit (e.g. 02 8D = 653 rpm) | ~10 per second | Matches the engine computer's own OBD answer (642–648 rpm at the same time) |
| Wheel speeds | 0x203 | Per Racelogic table above; to confirm while driving | ~11 per second | — |
| Steering angle | 0x003 | Per Racelogic table above; to confirm | ~20 per second | — |
| Brake | 0x005 | Per Racelogic table above; to confirm | ~10 per second | — |

Racelogic's "start bit 48" for engine speed corresponds to bytes 1–2 here, so its bit numbering counts from the other end; convert every Racelogic start bit the same way.

Also seen: both the engine computer (7E8) and the transmission computer (7E9) answer generic OBD requests for RPM (01 0C), coolant (01 05), load (01 04) and speed (01 0D). Readings must be taken from 7E8 only.

## Transmission data through the OBD port

The 722.9 transmission computer answers read-only KWP2000 requests (service 0x21, read data by local identifier) at 7E9 through the OBD port, so this works plug-and-play. Found with the hub's transmission scan (v4.3), key on, engine off, foot on brake, Nov 2, 2026 [3].

| Data | Request | Where | Values | Status |
|---|---|---|---|---|
| **Shifter position** | 21 3B | Byte 2 (second data byte) | ASCII letter: 0x50 "P", 0x52 "R", 0x4E "N", 0x44 "D" | **Confirmed** |
| Shifter position code (candidate) | 21 3A | Byte 4 | P 0x01, R 0x11, N 0x21, D 0x41 | Needs a second scan in a different order |
| Other ordinal candidates | 21 30 byte 8, 21 34 byte 9, 21 35 byte 1, 21 38 byte 3 | — | Step by one per shifter position | Scans were taken D, N, R, P, so a time counter would look the same; recheck |
| Current gear (1–7) | — | — | — | Needs a scan while driving |
| E/S drive program (candidate) | Broadcast 0x205 | Byte 2, bit 0 | 0x8D ↔ 0x8C when E/S is pressed (both raw captures 0032 and 0033) | Confirm with a 15-second hold test to tell a mode state from a button press |

Unchanging identifiers seen in every scan: 21 FA = "MT9500K0" (ASCII, likely the controller's part or software identifier).

## Sources

1. Daimler AG, WIS **GF00.19-P-0800FL**, *CAN electrical connector, as-built configuration*, Model 212.0 (18.12.08; printed 1/25/19).
2. Racelogic, *Mercedes-E-Class_W212_2010-2016.REF* (VBOX CAN database, "Racelogic Can Data File V1a").
3. W212 hub transmission scans, files 0027–0030 (labels D, N, R, P), 2011 E350 4MATIC, Nov 2, 2026.
