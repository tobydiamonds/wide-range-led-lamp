# WLED Setup Guide — Wide Range LED Lamp

## Flash WLED

Easiest method — use the web installer:
1. Go to **install.wled.me** in Chrome/Edge
2. Connect your D1 Mini via USB
3. Click **Install** → select **ESP8266** build
4. Follow the prompts

Or use the CLI:
```
pip install esptool
esptool.py --port COMx erase_flash
esptool.py --port COMx write_flash 0x0 WLED_x.x.x_ESP8266.bin
```

## Initial WiFi Setup

After flashing:
1. Connect to the WiFi AP: **WLED-AP** (password: `wled1234`)
2. Open **4.3.2.1** in your browser
3. Go to **WiFi Settings** and connect it to your home network

## LED Configuration

Go to **Config → LED Preferences**:

### Digital Outputs

| # | GPIO | Type | Length |
|---|------|------|--------|
| 1 | 2 | WS2811 (800kHz) | 10 * |
| 2 | 3 | WS2811 (800kHz) | 10 * |

\* 10 = addressable ICs per 1m strip. Adjust to match actual strip length (ICs, not individual LEDs).
For 3m synth wall shelves, use 30.

Set **Color Order** to **GRB** (standard for WS2811, but test — if colors are wrong, try RGB).

### Analog/PWM Outputs

| # | GPIO | Type |
|---|------|------|
| 3 | 4 | PWM White (1-channel) |
| 4 | 5 | PWM White (1-channel) |

## Segments

After saving LED config, go to the main control page:
- Segment 1: LEDs 0–9 → shelf 1 RGB
- Segment 2: LEDs 10–19 → shelf 2 RGB
- Segment 3: White output 1 → shelf 1 white
- Segment 4: White output 2 → shelf 2 white

You can name each segment (e.g., "Shelf 1 RGB", "Shelf 2 White") for easier control.

## Important Settings

- **Config → LED Preferences → Turn on "Use GPIO3 (RX) as LED output"** — this explicitly tells WLED to repurpose the RX pin
- **Config → LED Preferences → LED voltage**: set to 12V
- **Config → Sync Interfaces**: enable mDNS so you can reach it at `http://wled-xxxx.local`

## GPIO Pin Mapping

| GPIO | D1 Mini Label | Function |
|------|---------------|----------|
| 2 | D4 | RGB data strip 1 |
| 3 | RX | RGB data strip 2 |
| 4 | D2 | White PWM strip 1 |
| 5 | D1 | White PWM strip 2 |

## Notes

- After enabling GPIO3 as LED output, serial monitor won't receive data (flashing still works via USB/OTA)
- Use **OTA updates** going forward (Config → Security & Updates → upload .bin file)
- The WLED mobile app (iOS/Android) auto-discovers devices on your network
