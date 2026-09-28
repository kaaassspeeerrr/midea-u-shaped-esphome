# Reading the dongle logs

With the logger at DEBUG, `esphome logs <config>.yaml` shows every UART frame:

```
TX: AA 23 AC 8F 00 00 00 00 08 02 40 03 46 28 7F 7F 00 30 00 10 04 ...   <- SET_STATUS (0x40)
RX: AA 23 AC 00 00 00 00 00 08 03 C0 01 45 28 7F 7F 00 3C 00 10 04 ...   <- status reply (0xC0)
```

The body starts at the `40` (command) or `C0` (status) byte. Counting that
byte as 0, these are the ones that mattered on Midea U-shaped units:

| Byte | Meaning | Values seen |
|---|---|---|
| 1 | Power (bit 0) | `01` on, `00` off |
| 2 | Mode (bits 7–5) + setpoint (low nibble = °C − 16) | mode 2 cool, 3 dry, 5 fan only. `46` = cool 22 °C (72 °F), `44` = cool 20 °C (68 °F), `A4` = fan only |
| 3 | Fan speed | `28` (40) low, `3C` (60) medium, `50` (80) high as sent by ESPHome, `64` (100) high from the panel, `66` (102) auto |
| 7 | Swing | `30` off, `3C` vertical |
| 9 | ECO — status replies: bit `0x10`; commands: bit `0x80` (see below) | reply `10` on, `00` off |

**ECO is encoded differently in commands and replies.** In a status reply
(`C0`) ECO is byte 9 bit `0x10`. In a command (`40`) the library sets ECO with
byte 9 bit `0x80` — but it builds each command from a copy of the last status,
so a `0x10` left over from the reply goes out too. On these units a command
with `0x10` (and no `0x80`) turns ECO **off**; `0x80` turns it on; with
neither, a mode change into cool or dry gets the unit's default, ECO on.

The units also send unsolicited status broadcasts (`A0` frames), in which ECO
shows up as `0x80`/`0x90` in byte 9.

The status reply tells you what the AC actually accepted. That's how you can
see the unit switching ECO on by itself: the command has byte 9 = `00`, the
reply comes back with `10`.

Tips:
- `esphome logs` holds an API connection, which is harmless alongside Home
  Assistant.
- Log to a file and filter status replies:
  `grep -E "RX: AA 23 AC .* C0 " log.txt`
