[← Device list](../../README.md)

# Levoit LV600S / LUH-A602S-WEUR

Development support for the ultrasonic warm/cool-mist LV600S, using the original
ESP32-SOLO-1C. This is not the evaporative Superior 6000S. The previous private
implementation worked on this EU unit; **the new code here is not yet tested on
hardware**. Do not infer support for every LV600S variant.

- Module: ESP32-SOLO-1C / ESP32-S0WD, single core, 4 MB on the observed board.
- MCU UART: GPIO17 TX → MCU RX, GPIO16 RX ← MCU TX, **9600 8N1**.
- UART0 for flashing: GPIO1 TX, GPIO3 RX, GPIO0 strap, EN and GND. Main-board
  underside pads are not validated by a photo or module pin count.
- Disconnect mains and unknown power electronics before a serial connection.
  Use verified isolated power and 3.3 V logic. Do not attach a USB serial adapter
  to an open mains-powered appliance. Preserve two complete original backups,
  verify chip/protection state and retain its original partition table.
- This example uses ESPHome's default partition table for a **new installation**.
  It is not a drop-in OTA profile for the owner's already converted appliance.
  Existing installations must preserve their own original table and access keys.

Copy `secrets-example.yaml` to a private `secrets.yaml`; supply unique credentials.
The preview builder file pulls `main` until this component is released. Keep a
custom/pinned configuration for existing appliances with special partitions,
required OTA encryption or reset policy; do not replace one with the generic
password-based example. No flashing or resetting is performed by this repository.

Controls: power, display, Manual/Humidity/Humidity Sleep, target RH 40–80%, mist
1–9, warmth 0–3 and a timer up to 12 h with cancel. Original Auto and unknown mode
are display-only. Tank/water/fault/MCU diagnostics, humidity/temperature and the
component/model/version sensors support Home Assistant discovery. A native HA
humidifier entity and full original Auto parity are not implemented.

[Wire format, commands, safety and evidence](PROTOCOL.md).

Prior work: [wdm230/levoit-lv600s-esphome](https://github.com/wdm230/levoit-lv600s-esphome).
No source from that repository is bundled here. The component belongs to this
repository's MIT-licensed contribution; keep its protocol tests with changes.
