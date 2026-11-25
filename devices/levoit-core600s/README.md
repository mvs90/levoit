[← Back](../../README.md)

# Levoit Core 600S - Custom Firmware (ESPHome)

[EU owner live test with `1.0.09-tuct-review`](../../docs/esphome-devices/owner-test.md)

## Quick Facts

| Item | Value |
|------|-------|
| Model | Core 600S |
| Reported MCU FW | 2.0.1 and 1.0.12 (EU unit with earlier owner firmware) |
| ESP module | Check the board. The verified EU unit uses ESP32-SOLO-1C (ESP32-S0WD, 4 MB) |
| Fan Speeds | 4 |
| CADR (spec) | 641 m³/h |
| Noise | 26–54 dB |
| Room Size | 9–147 m² (97–1,582 ft²) |
| ESPHome | 2026.6.0+ |
| PM Sensor | Luftme LD15 |

## Features

| Feature | Type | Notes |
|---------|------|-------|
| Fan | fan | 4 speeds, presets: Manual / Auto / Sleep |
| Auto Mode | select | Default / Quiet / Room Size / ECO |
| Auto Mode Room Size | number | 9–147 m² |
| Display | switch | Toggle LED display |
| Child Lock | switch | |
| Light Detect | switch | Auto-dims display when ambient light is low |
| PM2.5 | sensor | µg/m³ |
| AQI | sensor | As reported by MCU |
| Current CADR | sensor | m³/h, updated every 5s |
| Filter Life Left | sensor | % remaining |
| Filter Low | binary_sensor | On when < 5% |
| Filter Lifetime | number | Configurable in months |
| Reset Filter Stats | button | Resets CADR/runtime counters |
| Timer | number | Run timer in minutes |
| MCU Version | text_sensor | |
| Error | text_sensor | "Ok" or "Sensor Error" |

## Teardown / Disassembly

Remove the base and filter from the upper assembly. Turn upside down.

![Levoit Core 600S Underside](images/C600S_underside_annotated.jpg)

Remove the screws highlighted in green. You can also remove the screws highlighted in blue to remove the fan grille and gasket piece to make removing the fan less unwieldy. In that case, the PM2.5 sensor will be lose so be careful.

![Levoit Core 600S Upper Inside](images/C600S_inside_upper_assembly.jpg)

To release the fan, remove the screws hidden behind plastic plugs in each of the handle.

![Levoit Core 600S Handle Screws](images/C600S_handle_screws.jpg)

Then pop the handles out and carefully slide the fan assembly out slightly. There are a couple of cables connecting to the control board which is clipped into the top fan grille. Luckily the holes left by the handles make it relatively easy to unplug these from the connectors on the control board.

![Levoit 600S Access Hole](images/C600S_handle_access_hole.jpg)

Carefully unplug these cables before pulling the fan assembly out.

![Levoit 600S Unplugging Cables](images/C600S_unplugging_cables.jpg)

That will get you access to flash the ESP32.

![Levoit 600S Top Case](images/C600S_top_case.jpg)

They were nice enough to silkscreen the ESP32 programming header on both sides so you don't even need to remove the board from the case if you don't want to. Make note of which ESP32 you have if you're not going to replace it. **ESP32-C3-SOLO-1** and **ESP32-SOLO-1C** have been found so far.

The header for the ESP32 is highlighted in blue. Connect your UART adapter here, making sure you get TX and RX the right way around. TX and RX on this board are TX and RX of the ESP32. Sometimes they're labeled to match your adapter. Note that it's **3V3** *not 5V*.

The header for the microvontroller is highlighted in yellow. This is not what you're looking for if you're looking to flash esphome.

![Levoit 600S Control Board](images/C600S_control_board_annotated.jpg)

## Install New ESP32 (Recommended)

> TODO: add wiring photos and pin mapping

## Flash Original ESP32

### Prerequisites

An EU `LAP-C601S-WEU` with MCU 1.0.12 ran ESPHome on its original
**ESP32-SOLO-1C**. It was identified as a single-core ESP32-S0WD with 4 MB flash
and used the original partition table. On that board the MCU UART runs at
115200 8N1, with GPIO17 as ESP TX and GPIO16 as ESP RX. The flash UART is
separate. Verify the chip and wiring for your board before choosing a config.

### Configure

Copy `secrets-example.yaml` to a local `secrets.yaml` and supply your Wi-Fi
and device keys. For the verified original SOLO-1C, use the original-module
config `levoit-core600s.yaml`. The C3 and S3 configs target replacement boards.
Keep the installed device's partition layout and credentials for later OTA.

### ESPHome Web Builder / Dashboard

Use the pre-generated builder yaml to flash without a local clone — all config is inlined, no `!include` or packages needed:

| File | Board |
|------|-------|
| `levoit-core600s-builder.yaml` | original ESP32-SOLO-1C |
| `levoit-core600s-builder-c3.yaml` | Seeed XIAO ESP32-C3 replacement |
| `levoit-core600s-builder-s3.yaml` | Seeed XIAO ESP32-S3 replacement |

Upload to the [ESPHome web builder](https://builder.esphome.io) or paste into the ESPHome dashboard. Regenerate with `.\make-builder-yaml.ps1` from the `devices/` folder.

### Backup Existing Firmware

```bash
esptool read_flash 0 ALL levoit-core600s-backup.bin
```

### Restore Original Firmware

```bash
esptool erase_flash
esptool write_flash 0x00 levoit-core600s-backup.bin
```

## Protocol Notes

Same UART protocol as Core 300S/400S with these differences:

- Status push: `CMD=01 40 41` (vs `B0 40` / `30 40` on 300S/400S)
- ACK uses `0x52` response type (not `0x12`) with trailing byte `0x01`
- Light Detect command: `CMD=01 E9 A5` PAY=`01`/`00`
- 4 Auto Mode options: Default (`00`), Quiet (`01`), Room Size (`02` + room_size LE), ECO (`03`)
- Status byte layout: display `[8]`, AQI `[11]`, PM2.5 `[12-13]`, child lock `[14]`, auto mode `[15]`, room size `[16-17]`, light detect `[21]`.
- The EU stock ESP requests full status with SEND `01 41 41` and no payload.
  Its MCU replies with `0x12 / 01 41 41`. The earlier owner firmware polled
  this every five seconds so a quiet device could still report current status.
- On the EU unit, payload byte 22 carries fault bits, including bit 0 (E1,
  service code 11802000) and bit 1 (E2, 11014000). The assembled unit reported
  zero in normal operation. A stale or truncated report does not establish that
  the device is fault-free. The periodical status query and raw fault reporting
  are implemented in the owner profile.
- Room size encoding: raw = sq_ft × 3.15 (MCU uses scaled sq ft)
