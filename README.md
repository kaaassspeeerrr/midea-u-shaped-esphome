# Midea U-shaped window AC + Home Assistant (ESPHome)

Control a **Midea U-shaped window air conditioner** from **Home Assistant**,
locally over your own Wi-Fi — no Midea app, no cloud — **with working fan
speed control in cool mode**.

Out of the box, a pre-flashed ESPHome dongle can switch the AC on and off,
change the mode and the temperature. But in cool mode, **changing the fan speed
from Home Assistant silently does nothing**. This repo fixes that. The short
reason is the AC's energy-saving **ECO** mode; the whole story, in plain
language, is in **[ECO explained](docs/eco-explained.md)**.

## Does this apply to me?

Yes, if you have:

| | Tested with |
|---|---|
| Air conditioner | Midea U-shaped window AC — **MAW10V1QWT** (10,000 BTU) and **MAW08V1QWT** (8,000 BTU). Other U-shaped models probably behave the same, but haven't been tested. |
| Dongle | **SMLIGHT SLWF-01Pro**, pre-flashed with ESPHome, plugged into the AC |
| Home Assistant | Any recent version, with the **ESPHome** integration |
| ESPHome | **2026.9.0** (see [Compatibility](#compatibility)) |

> **Safety first:** Midea U-shaped ACs were part of a 2025 recall (mold). Check
> your unit with Midea before anything else.

## What changes

| In Home Assistant, you… | Stock dongle firmware | With this repo |
|---|---|---|
| Turn the AC on/off, pick cool / dry / fan only | ✅ | ✅ |
| Change the temperature | ✅ | ✅ |
| Turn swing on/off | ✅ | ✅ — and it no longer resets your fan speed |
| Change fan speed in **fan only** | ✅ | ✅ |
| Change fan speed in **cool** | ❌ ignored | ✅ |
| See whether ECO is on, turn it on/off | ❌ invisible | ✅ ("Preset": `eco` / `none`) |
| Press "high" on the AC's own panel | shows as "auto" | shows as "high" |

## How it fits together

```mermaid
flowchart LR
    AC["Midea U-shaped AC"] -- "USB port<br/>behind the filter door" --- D["SLWF-01Pro dongle<br/>(runs ESPHome)"]
    D -- "your 2.4 GHz Wi-Fi" --- HA["Home Assistant<br/>(ESPHome integration)"]
    HA --- P["Your phone, dashboard,<br/>automations"]
```

The dongle is a tiny Wi-Fi computer (an ESP8266) that talks to the AC through
the USB port. **ESPHome** is the software running on it. This repo contains a
slightly modified version of ESPHome's Midea part; you install it by updating
the dongle's software once, over Wi-Fi.

## What you need

- The AC and the SLWF-01Pro dongle, plugged into the **USB port inside the AC,
  behind the filter door** (where Midea's own Wi-Fi stick would go). Turn the
  AC off before you open the filter door.
- The dongle on your Wi-Fi and showing up in Home Assistant. Pre-flashed
  dongles are set up following SMLIGHT's instructions for the SLWF-01Pro. The
  dongle only supports **2.4 GHz** Wi-Fi.
- The **ESPHome Device Builder** add-on in Home Assistant
  (Settings → Add-ons → Add-on Store → *ESPHome Device Builder*). It lets you
  edit the dongle's configuration and update it over Wi-Fi.

## Setup, step by step

1. **Open ESPHome Device Builder** (in the Home Assistant sidebar). Your dongle
   should be listed; if it offers **Take control** / **Adopt**, do that. You
   now have a configuration file (YAML) for the dongle — click **Edit**.
2. **Keep what's already there** at the top — especially the `api:` and `ota:`
   sections. They contain the keys your dongle already uses; if you replace
   them, Home Assistant and ESPHome can lose the connection to it.
3. **Add this block** (anywhere at the top level, e.g. after `esp8266:`). It
   tells ESPHome to use the modified Midea part from this repo:

   ```yaml
   external_components:
     - source: github://kaaassspeeerrr/midea-u-shaped-esphome@v1.0.0
       components: [midea]
   ```

4. **Make sure the UART and climate parts look like this** (replace the
   existing `uart:` and `climate:` sections if they differ):

   ```yaml
   uart:
     tx_pin: 12
     rx_pin: 14
     baud_rate: 9600

   climate:
     - platform: midea
       name: "AC"
       autoconf: false
       visual:
         temperature_step: 1
       supported_modes:
         - COOL
         - DRY
         - FAN_ONLY
       supported_swing_modes:
         - VERTICAL
       supported_presets:
         - ECO
         - BOOST
   ```

   A complete example file is in [examples/midea-u-shaped.yaml](examples/midea-u-shaped.yaml).

5. **Save, then Install → Wirelessly.** ESPHome downloads this repo, builds
   the new software and sends it to the dongle (a few minutes the first time).
   The dongle restarts; the AC keeps running.
6. **Check it worked** (next section).

## Check that it worked

1. In Home Assistant, open the AC. Below the fan and swing settings there
   should now be a **Preset** setting (`none`, `eco`, `boost`).
2. Set the AC to **cool**. After a moment the **ECO light on the AC turns on**
   by itself and the preset shows `eco`. That's normal (see
   [ECO explained](docs/eco-explained.md)).
3. Change the **fan speed** to low, then high. You should hear it change, and
   the ECO light stays on. With stock firmware, nothing would happen.

## Troubleshooting

| Problem | What to check |
|---|---|
| Fan speed still doesn't change in cool | Is there a **Preset** setting on the AC in Home Assistant? If not, the update didn't install — check the install log in ESPHome Device Builder. |
| Fan speed in **dry** is always auto | Normal — the AC itself forces auto fan in dry. |
| ECO keeps turning itself back on | Normal — the AC switches ECO on every time it enters cool. Turn it off with Preset → `none`. |
| Build fails after an ESPHome update | See [Compatibility](#compatibility). |
| Dongle shows "unavailable" | Check it's still in the USB port and on Wi-Fi. Giving it a fixed IP address in your router helps. |

## Going back to stock

Remove the `external_components:` block, keep the rest, and **Install**
again. Everything works as before, including the fan-speed limitation.

## Compatibility

The modified part is a copy of ESPHome **2026.9.0**'s Midea component. Newer
ESPHome versions will usually still build it, but if a future ESPHome changes
its internals the build can fail. If that happens, open an issue here.

## Presets

| Preset | Result |
|---|---|
| ECO | ✅ on/off from Home Assistant; fan speed works with it on |
| Boost | ✅ accepted and reported back by the AC (its effect on the unit wasn't measured) |
| Sleep | ❌ these units have no sleep mode — the AC ignores it and switches the fan to auto. Don't add `SLEEP` to `supported_presets`. |

## More detail

- [ECO explained](docs/eco-explained.md): what ECO does, what went wrong, what we changed and what we deliberately didn't
- [Test results](docs/test-results.md): every control, tested on both models
- [Advanced: reading the dongle's messages](docs/protocol.md), for developers
- [What was changed in the code](lib/MideaUART/PATCHES.md)

## Licence

GPLv3 (see [LICENSE](LICENSE)). Contains:

- `components/midea/`: from ESPHome 2026.9.0 (C++ GPLv3, Python MIT;
  © ESPHome). See [NOTICE](components/midea/NOTICE.md).
- `lib/MideaUART/`: from [dudanov/MideaUART](https://github.com/dudanov/MideaUART)
  (MIT, © Sergey Dudanov), with the patches listed in its PATCHES.md.

Not affiliated with Midea, SMLIGHT or ESPHome.
