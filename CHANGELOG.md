# Changelog

## v1.2.0 — 2026-09-28
- **ECO can be chosen from Home Assistant in dry and auto**, not just cool.
  Dry now always ends up with ECO on (it used to depend on the state before).
- Docs/example: `autoconf: true` (makes the display toggle work over USB),
  `min_temperature: 16 °C` (60 °F on the unit), display toggle and beeper
  extras, the AC's capability list, Follow Me is infrared-only.

## v1.1.0 — 2026-09-28
- **Sleep works from Home Assistant from any fan speed.** These units only
  accept sleep when the fan is already on auto; the dongle now switches the fan
  to auto first, then sends sleep. The fan stays on auto while sleep is on.
- Home Assistant shows **sleep** (not eco) when sleep and ECO are both on.
- Docs: warning to remove SMLIGHT's auto-update; Wi-Fi hotspot setup; what
  ECO and sleep do (from Midea's manual); boost marked undocumented; photos.

## v1.0.0 — 2026-09-28
- Fan speed works while ECO is on (ECO no longer forces auto fan).
- Panel/remote "high" (reported as 100) shows as high instead of auto.
