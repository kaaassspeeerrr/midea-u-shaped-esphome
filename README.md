# Midea U-shaped window AC + ESPHome

Local control of Midea U-shaped window air conditioners from Home Assistant
with ESPHome — no cloud — plus a small patch that makes **fan speed work in
cool mode**.

Tested on:

| Model | Capacity | Notes |
|---|---|---|
| MAW10V1QWT | 10,000 BTU | Full step-by-step test ([results](docs/test-results.md)) |
| MAW08V1QWT | 8,000 BTU | Full step-by-step test, same results ([results](docs/test-results.md)) |

Both are cooling-only (cool / dry / fan only), with vertical swing.

## The problem this fixes

With stock ESPHome, changing the fan speed from Home Assistant in cool mode
silently does nothing, and changing swing resets the fan to auto. The cause:

- The unit **switches ECO on by itself** every time it enters cool or dry.
- ESPHome's Midea library ignores fan-speed commands while any preset (ECO,
  sleep, turbo) is on.
- Without `supported_presets` in the config, Home Assistant can't see ECO at all.

The AC itself happily accepts fan speed with ECO on (the panel's fan button
works). This repo ships ESPHome's `midea` component with a patched copy of
the library that:

1. allows fan-speed changes while a preset is on (only Midea's AUTO mode
   still forces auto fan), and
2. reads the unit's "high" fan report (100) as high instead of auto.

Details: [lib/MideaUART/PATCHES.md](lib/MideaUART/PATCHES.md).

## Hardware

- **Dongle:** SMLIGHT **SLWF-01Pro** (ESP-12E / ESP8266), which talks to the
  AC over UART on GPIO12 (TX) / GPIO14 (RX) at 9600 baud.
- **Where it goes:** a female USB port inside the unit, behind the filter
  door (where Midea's own Wi-Fi stick would go). It's powered from that port.
- **Firmware:** the dongles ship pre-flashed with stock ESPHome. That works
  for mode, setpoint and swing, but has the ECO / fan-speed problem below and
  doesn't expose ECO. Reflash with this repo's config once (over Wi-Fi from
  the ESPHome dashboard or `esphome run`); later updates are OTA too.
- **Recall:** Midea U-shaped units were part of a 2025 CPSC recall for mold.
  Check yours with Midea before anything else.

## Setup

1. Copy [examples/midea-u-shaped.yaml](examples/midea-u-shaped.yaml) and
   [examples/secrets.yaml.example](examples/secrets.yaml.example) (as
   `secrets.yaml`) into your ESPHome config folder and fill in the secrets.
2. The example pulls the patched component straight from this repo:

   ```yaml
   external_components:
     - source: github://kaaassspeeerrr/midea-u-shaped-esphome@main
       components: [midea]
   ```

3. Flash (`esphome run midea-u-shaped.yaml`, or from the ESPHome
   dashboard). The dongle updates over Wi-Fi after the first flash.
4. In Home Assistant the climate entity gets a **preset** control:
   `none` / `eco` / `sleep` / `boost`. `none` = ECO off.

If you only want Home Assistant to *see* and clear ECO, adding
`supported_presets` to a stock ESPHome config is enough. The patch is what
lets you change fan speed while ECO stays on.

## Home Assistant notes

- Expect ECO to come back on whenever the unit switches to cool or dry. With
  this patch that no longer blocks fan speed, so you can simply leave it.
- Fan speed in dry mode is always auto — that's the unit, not ESPHome.
- Panel/remote changes show up in Home Assistant within about a second.

## Debugging

[docs/protocol.md](docs/protocol.md) explains how to read the dongle's UART
log and decode power, mode, setpoint, fan, swing and ECO from the status bytes.

## Licence

GPLv3 (see [LICENSE](LICENSE)). Contains:

- `components/midea/` — from ESPHome 2026.9.0 (C++ GPLv3, Python MIT;
  © ESPHome). See [NOTICE](components/midea/NOTICE.md).
- `lib/MideaUART/` — from [dudanov/MideaUART](https://github.com/dudanov/MideaUART)
  (MIT, © Sergey Dudanov), with the patches listed above.

Not affiliated with Midea, SMLIGHT or ESPHome.
