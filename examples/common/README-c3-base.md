# Seeed XIAO ESP32-C3 — C3 Base include (plain-English guide)

Note: this project uses a project-level `README.md` at the repository root. For the high-level overview and ESPHome Builder setup instructions, see `../README.md`.

This file documents the `Seeed xiao ESP32-c3 base.yaml` package used by device YAMLs in this repo. It explains each section and option in plain English for ESPHome builders so you can quickly understand, customize, and test C3 devices.

Summary

- Purpose: provide a reusable base configuration for Seeed XIAO ESP32‑C3 boards (esp32c3 variant) so device YAMLs can be small and device-specific.
- Typical usage: a device YAML uses `packages: device: !include "common/Seeed xiao ESP32-c3 base.yaml"` and provides substitutions like `device_name`, `friendly_name`, `api_key`, and `ota_password`.

Key sections and what they do

- `esp32`

  - `variant: esp32c3` and `board: seeed_xiao_esp32c3`: selects the C3 hardware and board definition.

  - `framework: esp-idf`: uses ESP-IDF framework.

- `esphome` -> `on_boot`:

  - Runs actions when the device boots. The base toggles GPIO2 output used for antenna selection (turning the external antenna on/off). If you add hardware that uses this pin, be aware of these actions.

- `logger`:

  - `level: DEBUG`, `baud_rate: 115200`, `hardware_uart: USB_CDC` — controls serial logging level and port. Change `level` to `INFO` or `WARN` on stable devices to reduce logs.

- `status_led`:

  - Configures the board LED pin (GPIO10, inverted). This shows device status on boot and at runtime.

- `api`:

  - `encryption.key: ${api_key}`: expects a substitution named `api_key` (provided by each device YAML or secrets). This enables encrypted API communications to Home Assistant.

  - `reboot_timeout`: auto-reboot behaviour if API is stuck.

- `ota`:

  - Uses `${ota_password}` for OTA updates. Device YAML should provide this substitution or you can use `!secret` values.

- `wifi`:

  - Domain and `use_address` set mDNS/host naming (e.g., `${device_name}.local`).

  - `fast_connect` and `enable_on_boot` are convenience flags for reconnect behaviour.

  - `ssid`/`password` use `!secret` in the base file — ESPHome Builder manages these automatically.

  - `ap` sets a fallback captive AP with its own `ssid` and `password` (`wifi_captive`).

- `captive_portal`, `mdns`:

  - Standard ESPHome helpers. `captive_portal` allows fallback setup; `mdns` enables local name discovery.

- Bluetooth / BLE

  - `esp32_ble_tracker` and `bluetooth_proxy`: enable BLE scanning and proxying. The base sets the BLE scan to `active: true`.

- `sensor` / `text_sensor` / `time`:

  - Adds common sensors: uptime (converted to hours), internal temperature, Wi‑Fi signal, Wi‑Fi info (BSSID, IP), and SNTP time source.

- `switch` (External Antenna)

  - A template switch controls GPIO2 output to toggle internal/external antenna. The output itself is defined in the `output:` section.

- `output`:

  - Defines the actual GPIO2 output used for antenna selection. If you repurpose this pin, update this section and the `switch` actions accordingly.

Substitutions and secrets

- The base file relies on substitutions passed in by device YAMLs: commonly `device_name`, `friendly_name`, `${api_key}`, and `${ota_password}`.

- ESPHome Builder automatically manages Wi-Fi secrets (`wifi_ssid`, `wifi_password`, `wifi_captive`) via its built-in secrets storage. Device examples in this repo may include inline dummy `api_key` and `ota_password` values for documentation — replace them before production use.

How device YAMLs use the base

- Example pattern (device YAML):

  ```yaml
  substitutions:
    device_name: esphomec3-garage
    friendly_name: Garage C3
    api_key: "..."
    ota_password: "..."

  esphome:
    name: ${device_name}
    friendly_name: ${friendly_name}

  packages:
    device: !include "common/Seeed xiao ESP32-c3 base.yaml"
  ```

Customization tips

- To change board-specific settings (pins, outputs, sensors), edit the base file only if the change applies to all devices that include it. Otherwise override or extend in the device YAML.

- If you need different antenna pin settings per device, remove antenna control from the base and define it per-device.

- Reduce `logger` level from `DEBUG` to `INFO` in production to reduce serial noise.

Using with ESPHome Builder

This package is designed for ESPHome Builder in Home Assistant, which automatically handles:
- Wi-Fi secrets storage
- Firmware compilation
- Device flashing and OTA updates

See the main `README.md` for complete ESPHome Builder setup instructions.

Notes and cautions

- The base uses `!secret` for Wi‑Fi values which ESPHome Builder manages automatically.

- The `api.encryption.key` and `ota.password` should be unique per device — ESPHome Builder can generate these for you.
