# Wide Range LED Lamp — Design Specification

## Overview

A LED lamp system using an ESP32 (NodeMCU 38-pin devkit) running WLED firmware to control both an RGB addressable strip and a warm white strip. The controller PCB is shared across lamp variants; only the power supply and strip lengths differ.

## LED Strips

### RGB Addressable Strip

- Type: WS2811 5050 RGB
- Voltage: DC 12V
- LED density: 30 LEDs/meter
- Addressing: 1 IC per 3 LEDs (10 addressable segments per meter)
- Power draw: ~7W per meter (full white)

### Warm White Strip

- Type: 5050/5054 SMD, non-addressable (2-wire)
- Voltage: DC 12V
- LED density: 60 LEDs/meter
- Power draw: ~12–14W per meter

## Controller

- MCU: ESP32 (Joy-IT SBC-NodeMCU-ESP32, 30-pin devkit)
- Firmware: WLED
- Form factor: NodeMCU plugs into 2×15 female pin headers on the carrier PCB
- Power: 5V pin fed from 12V→5V regulator on carrier board
- RGB output 1: GPIO26 → 330Ω series resistor → WS2811 data shelf 1
- RGB output 2: GPIO13 → 330Ω series resistor → WS2811 data shelf 2 (bodge wire from D13; GPIO33 failed RMT output)
- White output 1: GPIO27 → PWM via IRLB8721 N-channel MOSFET (shelf 1)
- White output 2: GPIO25 → PWM via IRLB8721 N-channel MOSFET (shelf 2)
- RGB signal: Direct 3.3V drive with 330Ω series resistor (no level shifter — tested OK with WS2811 at 3.3V)
- MOSFET: IRLB8721 (Vds: 30V, Id: 62A, Rds(on): ~8.7mΩ @ 4.5V Vgs)
  - Logic-level gate: fully on at 3.3V (compatible with ESP32 GPIO)
  - 100Ω gate resistor, 10kΩ pull-down

## Lamp Variants

### Ceiling Lamps (3× 65cm circular)

- Form factor: Circular, 65cm diameter (~2.04m circumference), 3 lamps total
- Strips per lamp: 1× warm white (~2m, ~120 LEDs) + 1× RGB addressable (~2m, ~60 LEDs, ~20 segments)
- Segments per lamp: 2 (warm white + RGB)
- Power budget per lamp: ~39–43W peak (white 24–28W + RGB 14W + ESP 0.5W on main only)
- Power supply:
  - Lamp 1 (main): 3× 20W LED drivers paralleled (60W, ~72% load at peak)
  - Lamps 2 & 3 (remote): 1× 60W LED driver each (local, to be purchased later)
- Interconnect: 4-wire cable from main lamp to each remote lamp (+12V, GND, RGB data, PWM)

#### Main board (installed at lamp 1)

- ESP32 + 7805 regulator (powered from lamp 1's 3× 20W drivers)
- Lamp 1 local: MOSFET (IRLB8721) + white strip connector, RGB data + strip connector
- All signal resistors for all 3 lamps:
  - 3× 330Ω series resistors (RGB data lines)
  - 3× 100Ω gate resistors (MOSFET PWM)
- Connector to daughter board: 2× PWM, 2× RGB data, GND (5-pin)
- Button on GPIO23

#### Daughter board (×1, installed at lamp 1 alongside main board)

- Handles lamps 2 & 3
- 5-pin input connector from main board (2× PWM, 2× RGB data, GND)
- 2× power inputs for 60W drivers (+12V, GND each)
- 2× MOSFET (IRLB8721) + 10K pull-down resistors
- 2× 4-wire output connectors to remote lamps (+12V, GND, RGB data, switched white)

#### Remote lamps (×2, lamps 2 & 3)

- 4-wire input from daughter board (+12V, GND, RGB data, switched white)
- Strips only, no electronics

### Synth Wall Lamp

- Configuration: 2 shelves × 3m each, 1 white + 1 RGB strip per shelf
- Strip totals: 6m white (360 LEDs) + 6m RGB (180 LEDs, 60 segments)
- Power budget: ~114–126W peak
- Power supply: TBD (e.g. Mean Well LPV-150-12 or equivalent)
- Notes: Power injection recommended at both ends of 3m runs to avoid voltage drop

## Power Summary

| Variant            | White (W) | RGB (W) | Total (W) | PSU              |
|--------------------|-----------|---------|-----------|------------------|
| Ceiling lamp 1     | 24–28     | 14      | 39–43     | 3× 20W parallel  |
| Ceiling lamp 2 & 3 | 24–28     | 14      | 39–43     | 1× 60W each      |
| Synth wall         | 72–84     | 42      | 114–126   | TBD ≥150W        |

## Design Tool

- PCB/Schematic: KiCad 10

## GPIO Pin Map

### Synth Wall Lamp

| GPIO | NodeMCU Pin | Function |
|------|-------------|----------|
| 26   | D26         | RGB data shelf 1 |
| 13   | D13         | RGB data shelf 2 (bodge wire; GPIO33 unusable) |
| 27   | D27         | White PWM shelf 1 |
| 25   | D25         | White PWM shelf 2 |
| 23   | GPIO23      | WLED button (active low, internal pull-up) |

### Ceiling Lamps

| GPIO | NodeMCU Pin | Function |
|------|-------------|----------|
| 26   | D26         | RGB data lamp 1 (local) |
| 13   | D13         | RGB data lamp 2 (remote) |
| 32   | D32         | RGB data lamp 3 (remote) — test early; GPIO33 failed RMT unexpectedly, fallback: GPIO16 or GPIO17 |
| 27   | D27         | White PWM lamp 1 (local) |
| 25   | D25         | White PWM lamp 2 (remote) |
| 14   | D14         | White PWM lamp 3 (remote) — brief flash at boot (JTAG TMS pull-up), cosmetic only |
| 23   | GPIO23      | WLED button (active low, internal pull-up) |

Note: All selected GPIOs support RMT (for addressable LEDs) and LEDC PWM (for white strips). GPIO0 is not exposed on the 30-pin board's headers (only accessible via onboard BOOT button), so GPIO23 is used for the external WLED button instead. GPIO 6–11 are reserved for internal flash and must not be used.

Caveats:
- GPIO32: Same silicon block as GPIO33 which failed RMT on the synth wall board for unknown reasons. Test RMT output early. If it fails, use GPIO16 or GPIO17 instead.
- GPIO14: JTAG TMS pin — has a pull-up at boot and outputs a brief PWM-like signal during ESP32 startup. Causes a short flash of the white strip on lamp 3 before WLED initialises. Cosmetic only, no functional impact.

## Resolved Decisions

- ESP module: Joy-IT SBC-NodeMCU-ESP32 (30-pin devkit, 2×15 headers)
- MOSFET: IRLB8721 (TO-220)
- RGB signal protection: 330Ω series resistor (no level shifter needed)
- Shelf connectors: 2× 4-pin (pin 1: 12V red, pin 2: RGB data white, pin 3: PWM grey, pin 4: GND brown)
- Voltage regulator: 7805 (12V → 5V for ESP32)

## WLED Configuration

### LED Outputs (Config → LED Preferences)

| Output | Type | GPIO | Color Order | Start | Length |
|--------|------|------|-------------|-------|--------|
| 1 | WS281x | 26 | BRG | 0 | 30 |
| 2 | PWM White | 27 | — | 30 | 1 |
| 3 | WS281x | 13 | BRG | 31 | 30 |
| 4 | PWM White | 25 | — | 61 | 1 |

### Segments

| Segment | Start | Stop | Function |
|---------|-------|------|----------|
| 0 | 0 | 30 | RGB shelf 1 |
| 1 | 30 | 31 | White shelf 1 |
| 2 | 31 | 61 | RGB shelf 2 |
| 3 | 61 | 62 | White shelf 2 |

### Boot preset

Save desired state as **Preset 0** (WLED applies preset 0 at boot by default).

## Ceiling Lamp Schematic Guide

### Main Board (new KiCad project: electrical-ceiling/)

Based on the synth wall schematic with these changes:

**Keep from synth wall:**
- ESP32 (U2, NodeMCU-ESP32-30pin) + 7805 (U1) + caps (C1–C4)
- Power input J1 (12V_IN, 2-pin terminal block)
- Button SW2 on GPIO23
- Q1 + R1 (100Ω gate) + R2 (10K pull-down) for WHITE_PWM_1 on GPIO27
- R4 (330Ω) for RGB_DATA_1 on GPIO26
- J2 (RGB_STRIP_1, 3-pin: +12V, data, GND)
- J3 (WHITE_STRIP_1, 2-pin: +12V, drain)

**Add:**
- R7 (330Ω) — series resistor for RGB_DATA_2 (GPIO13)
- R9 (330Ω) — series resistor for RGB_DATA_3 (GPIO32)
- R10 (100Ω) — gate resistor for WHITE_PWM_2 (GPIO25), output to daughter board
- R11 (100Ω) — gate resistor for WHITE_PWM_3 (GPIO14), output to daughter board
- J4 (TO_DAUGHTER, 5-pin header):
  - Pin 1: PWM_2 (after R10)
  - Pin 2: PWM_3 (after R11)
  - Pin 3: RGB_DATA_2 (after R7)
  - Pin 4: RGB_DATA_3 (after R9)
  - Pin 5: GND

**Remove (vs synth wall):**
- Q3, R5, R6 (2nd local MOSFET channel — moved to daughter board)
- R8 (2nd RGB 330Ω — replaced by R7 going to daughter board)
- J4 (WHITE_STRIP_2) and J5 (RGB_STRIP_2) — replaced by TO_DAUGHTER connector

### Daughter Board (new KiCad project: electrical-ceiling-daughter/)

**Components:**
- J1 (FROM_MAIN, 5-pin header): PWM_2, PWM_3, RGB_DATA_2, RGB_DATA_3, GND
- J2 (12V_IN_LAMP2, 2-pin terminal block): +12V, GND from 60W driver
- J3 (12V_IN_LAMP3, 2-pin terminal block): +12V, GND from 60W driver
- Q1 (IRLB8721): gate ← J1 pin 1 (PWM_2), drain → J4 pin 2, source → GND
- Q2 (IRLB8721): gate ← J1 pin 2 (PWM_3), drain → J6 pin 2, source → GND
- R1 (10K): Q1 gate pull-down to GND
- R2 (10K): Q2 gate pull-down to GND
- J4 (TO_LAMP2, 4-pin): +12V (from J2), GND, RGB_DATA_2 (from J1 pin 3), switched white (Q1 drain)
- J5 (TO_LAMP3, 4-pin): +12V (from J3), GND, RGB_DATA_3 (from J1 pin 4), switched white (Q2 drain)

**Connections:**
```
J1.1 (PWM_2) ──────────── Q1 gate
J1.2 (PWM_3) ──────────── Q2 gate
J1.3 (RGB_DATA_2) ──────── J4.3 (to lamp 2 strip data)
J1.4 (RGB_DATA_3) ──────── J5.3 (to lamp 3 strip data)
J1.5 (GND) ─────┬──────── Q1 source
                 ├──────── Q2 source
                 ├──────── R1 (pull-down)
                 ├──────── R2 (pull-down)
                 ├──────── J4.2 (lamp 2 GND)
                 └──────── J5.2 (lamp 3 GND)

J2.1 (+12V lamp 2) ─────── J4.1 (+12V to lamp 2)
J2.2 (GND lamp 2) ──────── GND bus
J3.1 (+12V lamp 3) ─────── J5.1 (+12V to lamp 3)
J3.2 (GND lamp 3) ──────── GND bus

Q1 drain ────────────────── J4.4 (switched white to lamp 2)
Q2 drain ────────────────── J5.4 (switched white to lamp 3)
```

## Open Questions

- Enclosure / mounting strategy per variant
- Power injection method for synth wall variant
