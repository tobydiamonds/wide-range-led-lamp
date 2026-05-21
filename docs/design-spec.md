# Wide Range LED Lamp — Design Specification

## Overview

A LED lamp system using an ESP32-C6 running WLED firmware to control both an RGB addressable strip and a warm white strip. The controller PCB is shared across lamp variants; only the power supply and strip lengths differ.

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

- MCU: ESP8266 (Wemos D1 Mini)
- Firmware: WLED
- Form factor: D1 Mini plugs into female pin headers on the carrier PCB
- Power: 5V pin fed from 12V→5V regulator on carrier board
- RGB output: GPIO2 (D4) → level shifter → WS2811 data (1-wire protocol)
- White output: GPIO4 (D2) → PWM via IRLB8721 N-channel MOSFET (analog dimming)
  - Vds: 30V, Id: 62A, Rds(on): ~8.7mΩ @ 4.5V Vgs
  - Logic-level gate: fully on at 3.3V (compatible with ESP8266 GPIO)

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

## Open Questions

- ~~ESP module selection~~ → Wemos D1 Mini (ESP8266, confirmed)
- ~~MOSFET selection for white strip PWM~~ → IRLB8721 (confirmed)
- Connector types for strip hookup
- Enclosure / mounting strategy per variant
- Power injection method for synth wall variant
