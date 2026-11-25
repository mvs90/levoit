# LV600S protocol notes

Observed hardware: LUH-A602S-WEUR, ESP32-SOLO-1C, MCU 1.0.56.
MCU UART: ESP TX17/RX16, 9600 baud, 8N1. UART0 for programming is separate.
The Superior 6000S uses a different model/protocol and is not an LV600S alias.

## Framing

`A5 control sequence length_lo length_hi checksum version command_lo command_hi result payload...`

Length counts all bytes after the checksum (four header bytes + payload). The
checksum is the complement of the sum of all other bytes, modulo 256. LV replies
use version 0/1 and result 0; SEND controls are 02/22, reply control 12. A 250 ms
inter-byte timeout and a 512-byte body bound prevent a damaged length from
holding later reports indefinitely. This component does not assume that the
purifiers' ACK policy applies to LV600S; the prior working LV implementation did
not send automatic report ACKs either.

## Status

4110 requests/returns status (16 or 20 bytes); 4111 pushes 20 bytes. Payload:

| Offset | Meaning |
|---|---|
| 0..2 | MCU patch, minor, major |
| 3 | Power |
| 4 | Tank state, zero = present |
| 5 | Water shortage |
| 6 | Display setting, 0/100 |
| 7 | Fog state; zero participates in the derived humidifying state |
| 8 | Target RH |
| 9 | Current RH |
| 10 | Temperature |
| 11 | Mode: 0 original Auto (read only), 1 Manual, 2 Humidity Sleep, 3 Humidity |
| 12 | Mist level 1..9 (internal scale; app groups into three levels) |
| 13 | Warm level 0..3, retained in standby |
| 14 | Uninterpreted diagnostic byte |
| 15 | MCU fault raw value; zero = no reported fault |
| 16..19 | Preserved raw diagnostics, no asserted meaning |

Heating-enabled and humidifying are **derived command states**, not measurements
of heater temperature or physical mist. Power, water, tank and fault state gate
both; warmth also requires a nonzero retained warm setting. Display is dark in
standby and Humidity Sleep regardless of the retained display setting. Status
older than 15 seconds is unavailable and cannot authorize an on/change command.

## Commands

| Command | Payload |
|---|---|
| A000 | Power 00/01 |
| A105 | Display 00/64 |
| A2E8 | 00, target RH 40..80 |
| 4113 | Mist 1..9, 00 00 00 |
| 4112 | Warm 0..3, 00 00 00 00 |
| A260 | Warm-enable, mist-enable, level (manual) |
| 4114 | RH target, 00 00 00 (humidity mode) |
| 4082 | RH target, 00 00 00 00 00 (sleep) |
| A264 | Timer seconds LE32, 0 cancels, max 43200 |
| A265 | Read remaining time; 2/4-byte LE reply |
| A266 | Report: remaining LE32, total LE32 |
| A129 | Wi-Fi pattern, on-ms LE16, off-ms LE16, repetitions |

A bounded eight-command queue confirms setters from later status reports, not
just ACKs. At most one retry for idempotent setters; timers are never retried.
Explicit off cancels queued setters and remains available with missing status.
Safety is checked again when a queued command is sent. Repeated timer snapshots
do not restart the host's countdown; expiry requires a zero from the MCU.

## Power-button events

Only physical SEND 02/22 with successful result, one-byte payload and the exact
opcode/action pair is emitted: D102/2 or D102/3 (pairing), D103/4 (reset request).
ACKs, foreign envelopes and D102/4 are rejected. The callback only reports the
event. Erasing NVS/opening a provisioning AP remains the owner's configuration
policy; this component never erases flash or changes credentials by itself.

## Evidence and limits

The byte layout and commands were compared with an owner's previously working
LV600S deployment and analysis of that owner's firmware. This new implementation
is independently written; it does not vendor the separate, unlicensed
wdm230/levoit-lv600s-esphome source. That project is credited as prior work.
The committed fixtures are synthetic, not raw private UART captures. New code is
host-tested and fully compiled; **it has not yet been installed or physically
validated on a humidifier**. Confirm it on hardware before calling it tested.
