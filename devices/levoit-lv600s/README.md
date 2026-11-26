[← Device list](../../README.md)

# Levoit LV600S / LUH-A602S-WEUR

Local ESPHome support for the ultrasonic warm/cool-mist LV600S, tested on one
EU `LUH-A602S-WEUR` with the original ESP32-SOLO-1C and owner firmware
`1.2.02-tuct-review`. [Live verification](../../docs/esphome-devices/owner-test.md)

- Module: ESP32-SOLO-1C / ESP32-S0WD, single core, 4 MB on the observed board.
- MCU UART: GPIO17 TX → MCU RX, GPIO16 RX ← MCU TX, **9600 8N1**.
- UART0 for flashing uses GPIO1 TX, GPIO3 RX and common GND. Bridge GPIO0 to
  GND to start the ESP32 ROM bootloader. Use an isolated external 5 V supply
  at the verified board input and a UART adapter with 3.3 V logic. Leave the
  adapter's VCC disconnected. [Disassembly and serial access](HARDWARE.md)
- This example uses ESPHome's default partition table for a **new installation**.
  It is not a drop-in OTA profile for the owner's already converted appliance.
  Existing installations must preserve their own original table and access keys.

Copy `secrets-example.yaml` to a private `secrets.yaml`; supply unique credentials.
The contribution targets upstream `main`. For a preview before integration,
use the LV600S example on the firmware fork's `codex/owner-test` branch.
Existing appliances with original partitions, required OTA encryption or the
owner's reset behaviour should use
the [secure owner profiles](https://github.com/mvs90/levoit-lv600s-local/tree/codex/tuct-alignment).

Controls: power, display, Manual/Humidity/Humidity Sleep, target RH 40–80%, mist
1–9, warmth 0–3 and a timer up to 12 h with cancel. Original Auto and unknown mode
are display-only. Tank/water/fault/MCU diagnostics, humidity/temperature and the
component/model/version sensors support Home Assistant discovery. A native HA
humidifier entity and full original Auto parity are not implemented.

[Wire format, commands, safety and evidence](PROTOCOL.md).
[Release review and dependencies](RELEASE-REVIEW.md).

Prior work: [wdm230/levoit-lv600s-esphome](https://github.com/wdm230/levoit-lv600s-esphome).
No source from that repository is bundled here. The component belongs to this
repository's MIT-licensed contribution; keep its protocol tests with changes.
