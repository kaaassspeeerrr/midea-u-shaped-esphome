# Origin

Copy of the `midea` component from ESPHome 2026.9.0
(`esphome/components/midea/`, https://github.com/esphome/esphome), under the
ESPHome License: C++ files GPLv3, Python files MIT. Copyright (c) 2019 ESPHome.

Only change: `climate.py` builds against the patched library in this repo's
`lib/MideaUART` instead of fetching MideaUART from GitHub (search for
`PATCH (midea-u-shaped-esphome)`).
