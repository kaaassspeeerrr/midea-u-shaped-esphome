# Changes from upstream MideaUART

Base: https://github.com/dudanov/MideaUART @ `eeea6c3e9b4474f067054592b435be1c4e466815`
(the commit pinned by ESPHome 2026.9.0). Removed from the copy: `.git`,
`.github`, `examples`, `test`. Search the source for `PATCH (midea-u-shaped-esphome`.

1. `src/Appliance/AirConditioner/AirConditioner.cpp`, `AirConditioner::control()`:
   upstream forces the fan to AUTO and drops fan-speed commands whenever a
   preset (ECO / SLEEP / TURBO) is active. Now only Midea AUTO mode does that.
   Midea U-shaped window units accept fan speed with ECO on (verified from
   the unit's own panel) and switch ECO on by themselves in cool.
2. `src/Appliance/AirConditioner/StatusData.cpp`, `StatusData::getFanMode()`:
   U-shaped units report the panel/remote "high" speed as 100, which upstream
   doesn't recognise (shown as AUTO). 100 now reads as HIGH.
