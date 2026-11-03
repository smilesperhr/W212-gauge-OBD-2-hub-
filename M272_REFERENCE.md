# M272 Reference Values

The values the health model and alerts are built on, and where each one came from. Every threshold here is an adjustable setting in the firmware, not hard-coded.

**Confidence:** **Mercedes** = from a Mercedes-Benz document · **Shop** = independent technician source · **Unverified** = AI search summary, to be checked against real healthy-car data before it's trusted.

## Fuel control

| Value | Number | Confidence | Source |
|---|---|---|---|
| Target air-fuel ratio (closed loop) | 14.7:1 (lambda 1.0) | Mercedes | [1] |
| Maximum fuel trim correction on ME engine computers | ±32%; at this limit a fault code sets and the check engine light comes on (P0170 series) | Mercedes | [1] |
| Short-term fuel trim | Should stay close to zero; resets at key-off | Mercedes | [1] |
| Long-term fuel trim zones | Learned separately for idle, lower part load and upper part load; no learning at full load | Mercedes | [1] |
| Healthy total trim (short + long term) | Within about ±5% (current rule); ±10% is the older, looser guideline | Shop | [2] |
| Short vs long term spread | No more than 10% apart | Shop | [2] |
| Generic OBD trims at cruise | Ideal near 0%; codes typically when long-term passes ±15–25% | Unverified | [4] |

## Airflow and idle

| Value | Number | Confidence | Source |
|---|---|---|---|
| MAF at idle, general | 2–7 g/s depending on engine size | Shop | [3] |
| MAF at warm idle, M272 | 3.0–5.0 g/s; under 2.5 g/s suggests a vacuum leak or skewed MAF | Unverified | [4] |
| MAF at 2,000 rpm, Park/Neutral, M272 | 7.5–9.0 g/s | Unverified | [4] |
| Warm idle speed, M272 | 550–650 rpm | Unverified | [4] |
| Intake manifold pressure at idle | About 35–40 kPa (if the ECU reports it) | Unverified | [4] |
| Fuel pressure | 50–55 psi / 3.8 bar (not readable over generic OBD) | Unverified | [4] |

## Vacuum leak signature

| Pattern | Confidence | Source |
|---|---|---|
| Trims positive at idle (low airflow, high vacuum, unmetered air has a large effect) | Shop | [2] |
| Trims return toward normal at cruise and full throttle as airflow rises | Shop | [2] |
| Model rule: idle zone trims high + upper part load zone near normal + idle MAF under ~2.5 g/s = "possible vacuum leak" | Built from [1], [2], [4] | — |

## M272 engine behavior the model must allow for

| Behavior | Detail | Confidence | Source |
|---|---|---|---|
| Intake runner flaps | Close (long runner) above 50% load from about 1,750 rpm; open (short runner) above 3,900 rpm. Airflow and trims can step at these points, so they are zone boundaries, not faults | Mercedes | [5] |
| Intake and swirl flaps | Moved by vacuum diaphragms (two intake actuators); swirl flap position read by Hall sensors B28/9 and B28/10. Diaphragms and their lines are leak suspects | Mercedes | [5] |
| Thermostat | Electrically assisted; the ECU moves the coolant target with load. Don't flag coolant changing on its own; watch warm-up time and overheating | Mercedes | [5] |
| EGR | None on the M272, so EGR is never a cause | Mercedes | [5] |
| Cam adjusters | Intake and exhaust each move 40°; four Hall cam sensors (B6/4–B6/7); left/right signals staggered 240° | Mercedes | [5] |
| Engine management | Bosch ME 9.7 (N3/10), hot film MAF B2/5, coolant sensor B11/4, crank sensor B70 | Mercedes | [5] |
| Output | 268 hp at 6,000 rpm, 258 lb-ft from 2,500–5,000 rpm, compression 10.7:1 | Mercedes | [5] |

## Leak suspects and mechanical notes

| Item | Detail | Confidence | Source |
|---|---|---|---|
| Crankcase ventilation hose at the centrifuge | The crankcase ventilation hose connects at the centrifuge (oil separator) cover on the cylinder head; the cover's seal and radial shaft seal are wear items to check and replace if damaged. A cracked hose or worn seal here lets unmetered air in | Mercedes | [6] AR01.20-P-2100EWS |
| Cam basic position | Checked at 55° crank angle before ignition TDC on cylinder 1 (305° mark on the belt pulley), and at 40° after TDC with the front cover off; pulse wheel markings must sit centered in the camshaft Hall sensor bores | Mercedes | [6] AR05.20-P-6010EW, AR05.10-P-7601EW |
| Timing chain rivet bolt | 32 Nm (reference value), engines 272 and 273 | Mercedes | [6] BA05.10-P-1007-01B |

The cam timing items matter for cam/crank correlation codes (P0016-P0019), which on the M272 can point to timing chain or balance shaft wear. Generic OBD doesn't report cam angle, so the hub can only flag those codes, not measure the timing.

## Sources

1. Mercedes-Benz StarTech, *Fuel Trim* (January 2015). https://www.mbwholesaleparts.com/content/dam/microsites/mb-wholesale-parts/pdf/1501-Jan-MBST-Fuel-Trim.pdf
2. Vehicle Service Pros, *Fuel Trim: Finding a Vacuum Leak and Performance Issues Using Fuel Trim Data* (2018). https://www.vehicleservicepros.com/industry-news/article/53089875/fuel-trim-finding-a-vacuum-leak-and-performance-issues-using-fuel-trim-data-2018-02-15
3. ALLDATA, *MAF Sensor Testing*. https://alldata.com/ca/en/support/tech-assist/article/maf-sensor-testing
4. Google AI search summary, "M272-specific idle airflow or trim chart" (Nov 3, 2026). Not a primary source; its Mercedes adaptation figures cited a VW/Audi tool and are left out here.
5. Mercedes-Benz, *M272 Engine Introduction* (student guide, SLK R171), via Mercedes-Benz Club NSK. https://nsk.mercedes-benz-clubs.com/wp-content/uploads/sites/7/2015/05/r171_motor-6-cyl-deel-1.pdf

6. Daimler AG, WIS repair procedures, *Engines 272, 273 in Model 212* (compiled PDF shared on MBWorld, 38 pages): AR05.10-P-7601EW Replace timing chain, AP15.10-P-1580EW Replace spark plugs, AR05.20-P-6992EW Remove/install camshafts, AR05.20-P-7201EWE Remove/install camshaft adjusters, AR01.30-P-5700EWL/EWR Front covers, AR01.20-P-2100EWS Remove/install centrifuge, AR01.20-P-5012EL/5013EL Cylinder head covers, AR05.10-P-8127EW Tensioning rail, AR05.10-P-7800EW Chain tensioner, AR05.20-P-6010EW Check basic position of camshafts. Mechanical procedures only; it contains no sensor values or diagnostic limits.

CAN network locations and W212 CAN signal IDs are in [W212_CAN_REFERENCE.md](W212_CAN_REFERENCE.md).
