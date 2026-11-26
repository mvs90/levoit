# LV600S release review

Scope is the tested EU `LUH-A602S-WEUR`, original ESP32-SOLO-1C, 4 MB flash
and MCU `1.0.56`. The [hardware guide](HARDWARE.md) documents the three
disassembly and wiring photos, external isolated 5 V at CN1, 3.3 V UART
logic, the GPIO0 boot strap, private backups and first-boot verification.
The [protocol notes](PROTOCOL.md) record the observed frame fields, status
layout, commands, service codes and evidence limits.

## Code and dependencies

| Area | Change | Effect on shared code |
|---|---|---|
| ESPHome `components/lv600s_humidifier` | Separate fixed-layout UART transport, bounded framing and write queue, MCU-confirmed controls, timer and diagnostics. The release candidate discards pending setters when status becomes stale or unsafe and retains explicit off | Uses ESPHome UART, sensor, binary-sensor and text-sensor APIs. Reads the existing component version helper. Its parser, queue and status layout are separate from the `levoit` purifier TLV implementation |
| Device YAML and builder generator | Adds the original-ESP sample and catalog entry. The generator uses the LV600S source package's Git URL and ref for its generated builder file | Other models keep the generator's original source/ref handling. Regenerating on the owner branch changes only the LV600S builder file |
| Tested secure owner profile | Preserves the original partition table, individual encrypted API and OTA access, stored Wi-Fi and owner reset policy. The mist, app-scale and warmth numbers read MCU values every second | These settings are in the LV600S profile of the owner repository. The original two purifier profiles use their own YAML and component |
| HA recognition and presentation | Adds the LV600S model and component, maps nested MCU diagnostic entities and German or English names, and renders dedicated controls. The release candidate excludes an auxiliary ESPHome fan from purifier controls and group discovery on a known LV600S | Existing device/entity IDs and the ESPHome transport are reused. Fan matching for other models is covered by a regression assertion. No custom humidifier platform is required |

The owner test branches also carry separate purifier UART/ACK validation and
HA template refresh protection for encrypted OTA. Those cross-model changes
are separate from the LV600S contribution branches. The HA owner profile uses
a custom local builder config, which avoids automatic template rewrites.

## Verification and scope

- The release candidate compiled with all LV600S entities under ESPHome
  2026.6.0 and 2026.10.2. The secure owner candidate also compiled with its
  original partition table, existing keys and required encrypted OTA.
  The current HA and upstream 0.6.8 contribution branches each passed 154
  pytest cases, Ruff lint and formatting, and the frontend state test.
- On ESPHome 2026.10.2 all seven Levoit and Philips comparison profiles
  compiled. Both LV600S example configurations resolved the published
  component from the owner fork and validated with a generated API key.

- The assembled owner device ran `1.2.01-tuct-review` and `1.2.02-tuct-review`.
  Local power, display, mist, warm level, humidity target, modes, countdown and
  timer cancel were confirmed. Firmware `1.2.02-tuct-review` reported mist,
  app scale and warm level to HA within 1.00–1.12 seconds. HA detected 20
  controls and diagnostics. [Owner test results](https://github.com/mvs90/levoit/blob/codex/owner-test/docs/esphome-devices/owner-test.md)
- Earlier hardware tests with the prior implementation established absent,
  empty and filled tank reporting, a water-shortage transition to standby,
  timer expiry and physical display response. The newer queue correction was
  exercised with synthetic stale and unsafe transitions, including recovery
  in the same receive burst and a pending off command.
- The release candidate uses the same entity names and command payloads.
  Original Auto is read-only. Heating and humidifying flags are derived from
  MCU status. Long-term behaviour, other board and MCU revisions, and the
  newest builds' fault and reset handling still need a device run.
- The generically generated builder sample and the owner's secure profile
  serve different installation paths. Keep the secure profile for the already
  converted device. [Secure owner build and live verification](https://github.com/mvs90/levoit-lv600s-local/blob/codex/tuct-alignment/docs/UPSTREAM-ALIGNMENT.md)
