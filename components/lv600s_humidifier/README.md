# LV600S humidifier (development)

Separate fixed-layout component for LV600S, 9600 baud. The `levoit` purifier/TLV
component and the Superior 6000S model are unchanged.

See the [device guide](../../devices/levoit-lv600s) and
[protocol notes](../../devices/levoit-lv600s/PROTOCOL.md). The parent block supports
optional sensors, binary sensors and text sensors; controls use ordinary ESPHome
template entities from the example. It does not create a fake fan or filter.
Its `model: LV600S` and `esp_version` use the shared repository version source.

Validation: standalone C++ protocol/queue/state tests with synthetic frames,
and a full ESP32 single-core ESPHome 2026.10.2 build with every optional entity.
Physical validation of this new implementation is pending. CI additionally builds
at the project's minimum ESPHome 2026.6.0 and newest version.

```sh
python components/lv600s_humidifier/tests/test_protocol.py
esphome compile components/lv600s_humidifier/tests/test-all-entities.yaml
```

The default native test enables ASan+UBSan. Set `LEVOIT_TEST_SANITIZERS=undefined`
only on hosts where the ASan runtime itself is broken; do not describe that as an
ASan success. The test never contacts a device.

## Unreleased

Initial LV600S implementation. No release version bump pending maintainer review.
