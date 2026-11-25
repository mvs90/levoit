# Owner test results for three EU devices

Live verification on 25 November 2026 used the original Wi-Fi modules and
existing Home Assistant ESPHome entries. The exact component source was
`13f7ff6e6fee448158400fab8d5ab4c189dada64`. The
[owner profiles and local patches](https://github.com/mvs90/levoit-lv600s-local/blob/codex/tuct-alignment/docs/UPSTREAM-ALIGNMENT.md)
record the image hashes, original partition layouts, secure build settings and
the reproduction procedure. The generic device examples are for separate builds.

| Device | Owner firmware | Live checks |
|---|---|---|
| [Core 600S](../../devices/levoit-core600s/) | `1.0.09-tuct-review`, MCU `1.0.12` | Local power, two manual fan levels, display off/on with fan running, timer start/cancel and fresh MCU status with no reported fault |
| [Vital 100S](../../devices/levoit-vital100s/) | `1.0.06-tuct-review`, MCU `1.0.5` | Local power, manual 50% and 75% with MCU feedback, display off/on and timer start/cancel. The repeat test passed with individual confirmations between mode and speed changes |
| [LV600S](../../devices/levoit-lv600s/) | `1.2.02-tuct-review`, MCU `1.0.56` | Local power, display, mist/warm levels, humidity target, Manual/Humidity/Sleep, countdown and cancel. HA state events for mist, its app scale and warm level arrived 1.00–1.12 s after the command |

The LV600S countdown was verified on `1.2.01-tuct-review`. The subsequent
change to `1.2.02-tuct-review` covered the three level entities' polling
intervals. The test measured HA state events for MCU-reported values. Current
firmware versions were checked independently via encrypted ESPHome API.

All three owner images were installed over required encrypted OTA and kept
their original partition layouts and individual keys. All corresponding
configurations compiled in the ESPHome Device Builder through the
[tested HA fork](https://github.com/mvs90/levoit-esphome-ha/tree/codex/owner-test).
The tests used digital state feedback. The earlier hands-on hardware findings are
in each device guide. This run did not exercise physical buttons or factory reset.
