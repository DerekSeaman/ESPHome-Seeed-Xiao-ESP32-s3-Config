# Seeed XIAO ESP32-S3 — S3 Base include (plain-English guide)

Note: this project uses a project-level `README.md` at the repository root. For the high-level overview and ESPHome Builder setup instructions, see `../README.md`.

This file documents the `Seeed xiao ESP32-s3 base.yaml` package used by device YAMLs in this repo. It explains each section and option in plain English for ESPHome builders so you can quickly understand, customize, and test S3 devices.

Summary

- Purpose: provide a reusable base configuration for Seeed XIAO ESP32-S3 boards (esp32s3 variant) so device YAMLs can be small and device-specific.
- Typical usage: a device YAML uses `packages: device: !include "common/Seeed xiao ESP32-s3 base.yaml"` and provides substitutions like `device_name`, `friendly_name`, `api_key`, and `ota_password`.

Key sections and what they do

- `esp32`

  - `variant: esp32s3` and `board: seeed_xiao_esp32s3`: selects the S3 hardware and board definition.

  - `framework: esp-idf`: uses ESP-IDF framework.

- `esphome`:

  - `name: ${device_name}` and `friendly_name: ${friendly_name}`: the base config now includes these, so device YAMLs no longer need to define the `esphome:` section — just provide the substitutions.

- `logger`:

  - `level: DEBUG`, `baud_rate: 115200`, `hardware_uart: USB_SERIAL_JTAG` — controls serial logging level and port. The S3 uses USB_SERIAL_JTAG for native USB serial. Change `level` to `INFO` or `WARN` on stable devices to reduce logs.

- `status_led`:

  - Configures the board LED pin (GPIO21). This shows device status on boot and at runtime.

- `api`:

  - `encryption.key: ${api_key}`: expects a substitution named `api_key` (provided by each device YAML or secrets). This enables encrypted API communications to Home Assistant.

  - `reboot_timeout`: auto-reboot behaviour if API is stuck.

- `ota`:

  - Uses `${ota_password}` for OTA updates. Device YAML should provide this substitution or you can use `!secret` values.

- `wifi`:

  - Domain and `use_address` set mDNS/host naming (e.g., `${device_name}.local`).

  - `power_save_mode: NONE`: disables WiFi power saving for better BLE performance and more consistent connectivity.

  - `fast_connect` and `enable_on_boot` are convenience flags for reconnect behaviour.

  - `ssid`/`password` use `!secret` in the base file — ESPHome Builder manages these automatically.

  - `ap` sets a fallback captive AP with its own `ssid` and `password` (`wifi_captive`).

  - `on_disconnect`: increments a counter (`_wifi_disconnects_since_boot`) each time Wi-Fi disconnects, exposed via a template sensor for diagnostics.

- `captive_portal`, `mdns`:

  - Standard ESPHome helpers. `captive_portal` allows fallback setup; `mdns` enables local name discovery.

- Bluetooth / BLE

  - `esp32_ble_tracker` and `bluetooth_proxy`: enable BLE scanning and proxying for Home Assistant presence detection.

  - BLE scan parameters are configurable via a `select` entity with two profiles:
    - **Aggressive** (default): 160ms interval, 160ms window (100% duty cycle) — maximum presence detection accuracy
    - **Balanced**: 320ms interval, 160ms window (50% duty cycle) — reduced power consumption
    - Profile selection persists across reboots via `restore_value: true`

- `sensor` / `text_sensor` / `time` / `globals`:

  - Adds common sensors: uptime (converted to hours), internal temperature, Wi-Fi RSSI, Wi-Fi info (BSSID, IP, SSID, MAC), Wi-Fi disconnects (since boot), and SNTP time source.

  - `globals`: defines `_wifi_disconnects_since_boot` counter (not restored on reboot) tracked by the Wi-Fi `on_disconnect` handler.

- `button`:

  - A restart button to reboot the device from Home Assistant.

Substitutions and secrets

- The base file relies on substitutions passed in by device YAMLs: commonly `device_name`, `friendly_name`, `${api_key}`, and `${ota_password}`.

- ESPHome Builder automatically manages Wi-Fi secrets (`wifi_ssid`, `wifi_password`, `wifi_captive`) via its built-in secrets storage. Device examples in this repo may include inline dummy `api_key` and `ota_password` values for documentation — replace them before production use.

How device YAMLs use the base

- Example pattern (device YAML):

  ```yaml
  substitutions:
    device_name: esphomes3-garage
    friendly_name: Garage S3
    api_key: "..."
    ota_password: "..."

  packages:
    device: !include "common/Seeed xiao ESP32-s3 base.yaml"
  ```

  Note: The `esphome:` section is no longer needed in device YAMLs — the base config now includes `name:` and `friendly_name:` using the substitutions you provide.

Customization tips

- To change board-specific settings (pins, outputs, sensors), edit the base file only if the change applies to all devices that include it. Otherwise override or extend in the device YAML.

- Reduce `logger` level from `DEBUG` to `INFO` in production to reduce serial noise.

Using with ESPHome Builder

This package is designed for ESPHome Builder in Home Assistant, which automatically handles:
- Wi-Fi secrets storage
- Firmware compilation
- Device flashing and OTA updates

See the main `README.md` for complete ESPHome Builder setup instructions.

Notes and cautions

- The base uses `!secret` for Wi-Fi values which ESPHome Builder manages automatically.

- The `api.encryption.key` and `ota.password` should be unique per device — ESPHome Builder can generate these for you.
