# Test results

Tested 2026-09-28 on both models, each with an SMLIGHT SLWF-01Pro running
ESPHome 2026.9.0 (both bought March 2025):

- **MAW10V1QWT** (10,000 BTU): stock firmware first, then with this repo.
- **MAW08V1QWT** (8,000 BTU): with this repo (v1.0.0). Every row below
  confirmed on the unit. Same results — both models behave identically.

Method: one change at a time from Home Assistant, someone watching the unit,
and the dongle's UART traffic logged with `esphome logs` so each command can
be checked against the AC's own status reply (see [protocol.md](protocol.md)).

## Home Assistant → AC

| Control | Stock ESPHome | This repo | Notes |
|---|---|---|---|
| Off / on | ✅ | ✅ | |
| Cool | ✅ | ✅ | The unit turns **ECO on by itself** when it enters cool, even though the command says ECO off |
| Dry | ✅ | ✅ | Fan is forced to auto by the unit (normal for Midea). ECO depends on the state before — see below |
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

## ECO when switching into dry

Tested on both models at the same time, identical results:

| ECO before (in cool) | After switching to dry |
|---|---|
| on | **off** |
| off | **on** |

It looks like a flip but isn't: the library treats ECO as cool-only, so from
cool+ECO it sends dry *without* a preset — and the command still carries the
status bit `0x10` copied from the last status reply, which these units treat
as "ECO off". From cool without ECO that bit isn't there, and the unit applies
its default for a mode change: ECO on. See [protocol.md](protocol.md).

Similarly, dry (with ECO on) → cool keeps ECO on: the library carries the
preset over by sending the mode change, then a second command with ECO.

This repo leaves that behaviour alone; fan speed works either way.

## Why fan speed "doesn't work" on stock ESPHome

1. These units switch ECO on by themselves whenever they go into cool (and
   into dry, unless ECO was on just before).
2. Upstream MideaUART's `AirConditioner::control()` forces the fan to AUTO and
   drops fan-speed commands while any preset (ECO, SLEEP, TURBO) is active.
3. Without `supported_presets`, Home Assistant can't even see that ECO is on.

So in cool mode fan-speed changes from Home Assistant do nothing, and changing
swing resets the fan to auto. The AC itself is fine with it: pressing the fan
button on the panel with ECO on changes the speed and ECO stays on (confirmed
from the unit's status replies: ECO flag stays set while fan goes 40 → 60 →
100 → auto).
