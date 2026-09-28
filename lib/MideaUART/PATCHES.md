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
3. `src/Appliance/AirConditioner/AirConditioner.cpp`, `AirConditioner::control()`
   (v1.1.0): SLEEP keeps upstream's "fan to auto, ignore fan changes" rule —
   these units only accept sleep when the fan is **already** on auto (fan auto +
   sleep in one command is refused). When sleep is requested with the fan not
   on auto, the dongle now sends two commands: fan auto first, then sleep
   (reusing upstream's two-command path for mode changes).
4. `src/Appliance/AirConditioner/StatusData.cpp`, `StatusData::getPreset()`
   (v1.1.0): SLEEP is checked before ECO. These units can run both at once;
   upstream reported only ECO, hiding sleep set from the panel or remote.
5. `src/Appliance/AirConditioner/AirConditioner.cpp`, `checkConstraints()`
   (v1.2.0): ECO is allowed in DRY and AUTO as well as COOL (Midea's manual
   lists Energy Saver for all three). Side effect: ECO carries over from cool
   into dry.
