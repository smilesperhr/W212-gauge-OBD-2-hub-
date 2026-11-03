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

## Sources

1. Daimler AG, WIS **GF00.19-P-0800FL**, *CAN electrical connector, as-built configuration*, Model 212.0 (18.12.08; printed 1/25/19).
2. Racelogic, *Mercedes-E-Class_W212_2010-2016.REF* (VBOX CAN database, "Racelogic Can Data File V1a").
