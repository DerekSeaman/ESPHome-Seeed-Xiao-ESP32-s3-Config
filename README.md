# Seeed Studio XIAO ESP32‑S3 ESPHome Device Builder Package

This repository contains a reusable **ESPHome Device Builder package** for the Seeed Studio XIAO ESP32‑S3 (esp32s3) boards. Several configurations are provided for you to choose from. Each configuration is covered below. These are designed to work with ESPHome Device Builder 2026.7 and later.

Quick overview

- Layout:

  - `examples/Seeed XIAO ESP32-s3 base.yaml` — S3 base configuration (board, wifi, sensors, status LED, etc.), no Bluetooth proxy

  - `examples/Seeed XIAO ESP32-s3 IRK.yaml` — S3 board specifics designed to be used with my IRK Capture package (see below)

  - `examples/Seeed XIAO ESP32-s3 proxy.yaml` — S3 base configuration with customizable Bluetooth proxy functionality

  - `examples/Seeed XIAO ESP32-s3 remote.yaml` — Package definition designed to be used with a generic ESPHome Device Builder S3 device configuration. This will reference one of the above configurations and dynamically pull it in at compile time.

![Seeed XIAO ESP32-S3 PCB](docs/Seeed%20s3%20pcb.jpg)

**Key feature:** The XIAO ESP32-S3 has native USB and the most onboard resources (RAM/flash/PSRAM) in the Seeed XIAO ESP32 lineup, with a status LED for at-a-glance device state. No RF antenna switch on this board — just a single external antenna.

## Using with ESPHome Device Builder

This is an **ESPHome Device Builder package** designed to work seamlessly with the ESPHome Device Builder tool in Home Assistant. This has been tested with ESPHome Device Builder 2026.7.4. Follow these steps to create a new device with the custom Seeed Studio XIAO ESP32-S3 configuration:

1. Install the **ESPHome Device Builder** add-on from the Home Assistant Add-on Store
2. Go into the **ESPHome Device Builder** and in the upper right click on **+ Create device**
3. Select **Create new project**
4. Click on **ESP32-S3**, then type **Seeed** in the search boards field
5. Click **+ Select** on the **Seeed Studio XIAO ESP32S3** card
6. Enter a device name, click **Finish Setup**
7. Paste the contents of the S3 remote file to the bottom of your ESPHome Device Builder template [S3 Remote File](https://github.com/DerekSeaman/ESPHome-Seeed-Xiao-ESP32-s3-Config/blob/main/examples/Seeed%20XIAO%20ESP32-s3%20remote.yaml)
8. Depending on which version you want, modify **file:** as needed (proxy, base, IRK)
9. Modify any other settings as needed, then install to your Seeed Studio XIAO ESP32-S3 device.

Your configuration should look something like this, except your `ref:` should point to **main**.

![YAML Example](docs/YAML-config.jpg)

## IRK Configuration Details

I built a special S3 IRK configuration that is designed to be used with my IRK Capture package for ESPHome. It can be found at: [DerekSeaman/irk-capture](https://github.com/DerekSeaman/irk-capture). This eliminates some of the duplicate settings already built into my IRK Capture package and only adds the unique settings needed for the Seeed Studio XIAO ESP32-S3.

## Bluetooth Proxy

If you use the **proxy** configuration, your S3 will act as a Bluetooth proxy with three selectable BLE scan profiles:

- **Low**: 200ms interval, 18.75ms window (9% duty cycle) — minimal power consumption
- **Medium** (default): 200ms interval, 56.25ms window (28% duty cycle) — balanced performance
- **High**: 200ms interval, 100ms window (50% duty cycle) — maximum presence detection accuracy

Profile selection persists across reboots. If you are using the proxy with room-level presence detection, medium or high is recommended. Otherwise, low should be sufficient and will use less Wi-Fi bandwidth.

![BLE Scanner Profiles](docs/BLE-proxy.jpg)

## Status LED Patterns

The onboard LED (GPIO21, active LOW) provides visual feedback about the device state:

| Pattern | Meaning |
|---------|---------|
| Solid ON | Everything OK - WiFi connected, API connected with active client |
| Slow blink (~1Hz) | Warning - WiFi connected but API client not connected/subscribed |
| Fast blink (~2-3Hz) | Error - No WiFi connection |
| Very fast blink (~10Hz) | Critical error during boot or OTA in progress |

## ESPHome Device Page

Here's what the proxy device looks like in Home Assistant's ESPHome integration:

![ESPHome Device Page](docs/screenshot-1.jpg)

The device page shows:

- **Device info**: Board type, firmware version, and MAC address
- **Controls**: BLE Scan Profile selector (Low/Medium/High)
- **Configuration**: Firmware management and OTA updates
- **Diagnostic**: BSSID, internal temperature, IP address, MAC address, SSID, uptime, Wi-Fi Channel, Wi-Fi disconnects (since boot), and Wi-Fi RSSI
