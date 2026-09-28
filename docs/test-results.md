# Test results

Tested 2026-09-28 on a **MAW10V1QWT** (10,000 BTU, bought March 2025) with an
SMLIGHT SLWF-01Pro running ESPHome 2026.9.0. A **MAW08V1QWT** (8,000 BTU, same
purchase) runs the same firmware and behaves the same in daily use; it hasn't
been put through this step-by-step test yet.

Method: one change at a time from Home Assistant, someone watching the unit,
and the dongle's UART traffic logged with `esphome logs` so each command can
be checked against the AC's own status reply (see [protocol.md](protocol.md)).

## Home Assistant → AC

| Control | Stock ESPHome | This repo | Notes |
|---|---|---|---|
| Off / on | ✅ | ✅ | |
| Cool | ✅ | ✅ | The unit turns **ECO on by itself** when it enters cool, even though the command says ECO off |
| Dry | ✅ | ✅ | Fan is forced to auto by the unit (normal for Midea); ECO turns on by itself |
| Fan only | ✅ | ✅ | ECO clears |
| Setpoint | ✅ | ✅ | Display updates, compressor starts (after the usual ~3 min protection delay) |
| Fan speed (ECO off) | ✅ | ✅ | low / medium / high |
| Fan speed (ECO on) | ❌ silently ignored | ✅ | See below |
| Swing (vertical) on/off | ✅ | ✅ | |
| Swing while ECO on | ⚠️ also resets fan to auto | ✅ fan kept | Same cause |
| ECO on/off | ❌ not exposed | ✅ | Needs `supported_presets` in the YAML |

## AC panel/remote → Home Assistant

| Change on the unit | Stock ESPHome | This repo |
|---|---|---|
| Fan low / medium | ✅ | ✅ |
| Fan high | ⚠️ shows as **auto** (unit reports 100) | ✅ high |
| Swing, setpoint, mode | ✅ | ✅ |
| ECO on/off | ❌ invisible | ✅ preset |

## Why fan speed "doesn't work" on stock ESPHome

1. These units switch ECO on by themselves whenever they go into cool or dry.
2. Upstream MideaUART's `AirConditioner::control()` forces the fan to AUTO and
   drops fan-speed commands while any preset (ECO, SLEEP, TURBO) is active.
3. Without `supported_presets`, Home Assistant can't even see that ECO is on.

So in cool mode fan-speed changes from Home Assistant do nothing, and changing
swing resets the fan to auto. The AC itself is fine with it: pressing the fan
button on the panel with ECO on changes the speed and ECO stays on (confirmed
from the unit's status replies: ECO flag stays set while fan goes 40 → 60 →
100 → auto).
