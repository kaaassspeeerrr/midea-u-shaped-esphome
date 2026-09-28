# Changelog

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
