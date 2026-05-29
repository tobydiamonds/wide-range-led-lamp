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
- RGB output 2: GPIO33 → 330Ω series resistor → WS2811 data shelf 2
- White output 1: GPIO27 → PWM via IRLB8721 N-channel MOSFET (shelf 1)
- White output 2: GPIO25 → PWM via IRLB8721 N-channel MOSFET (shelf 2)
- RGB signal: Direct 3.3V drive with 330Ω series resistor (no level shifter — tested OK with WS2811 at 3.3V)
- MOSFET: IRLB8721 (Vds: 30V, Id: 62A, Rds(on): ~8.7mΩ @ 4.5V Vgs)
  - Logic-level gate: fully on at 3.3V (compatible with ESP32 GPIO)
  - 100Ω gate resistor, 10kΩ pull-down

## Lamp Variants

### Ceiling Lamp

- Strip length: ~1m white + ~1m RGB
- Power budget: ~19–21W peak
- Power supply: Goobay 20W LED transformer, 12V DC output
- Addressable segments: 10

### Synth Wall Lamp

- Configuration: 2 shelves × 3m each, 1 white + 1 RGB strip per shelf
- Strip totals: 6m white (360 LEDs) + 6m RGB (180 LEDs, 60 segments)
- Power budget: ~114–126W peak
- Power supply: TBD (e.g. Mean Well LPV-150-12 or equivalent)
- Notes: Power injection recommended at both ends of 3m runs to avoid voltage drop

## Power Summary

| Variant       | White (W) | RGB (W) | Total (W) | PSU           |
|---------------|-----------|---------|-----------|---------------|
| Ceiling (1m)  | 12–14     | 7       | 19–21     | Goobay 20W    |
| Synth wall    | 72–84     | 42      | 114–126   | TBD ≥150W     |

## Design Tool

- PCB/Schematic: KiCad 10

## GPIO Pin Map

| GPIO | NodeMCU Pin | Function |
|------|-------------|----------|
| 26   | D26         | RGB data shelf 1 |
| 33   | D33         | RGB data shelf 2 |
| 27   | D27         | White PWM shelf 1 |
| 25   | D25         | White PWM shelf 2 |
| 23   | GPIO23      | WLED button (active low, internal pull-up) |

Note: All selected GPIOs are free of boot-mode side effects and support RMT (for addressable LEDs) and LEDC PWM (for white strips). GPIO0 is not exposed on the 30-pin board's headers (only accessible via onboard BOOT button), so GPIO23 is used for the external WLED button instead. GPIO 6–11 are reserved for internal flash and must not be used.

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
| 1 | WS281x | 26 | BRG | 0 | 50 |
| 2 | PWM White | 27 | — | 50 | 1 |

(Shelf 2 would add outputs 3+4 on GPIO33 and GPIO25 with appropriate start offsets)

### Segments

| Segment | Start | Stop | Function |
|---------|-------|------|----------|
| 0 | 0 | 50 | RGB shelf 1 |
| 1 | 50 | 51 | White shelf 1 |

### Boot preset

Save desired state as **Preset 0** (WLED applies preset 0 at boot by default).

## Open Questions

- Enclosure / mounting strategy per variant
- Power injection method for synth wall variant
