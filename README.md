# AirCast

**Open-source air quality monitoring device**  
Real-time environmental sensing with Blynk dashboard and Discord webhook alerts.

---

## Table of Contents

- [Overview](#overview)
- [Hardware](#hardware)
- [Sensors](#sensors)
- [Features](#features)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Firmware Setup](#firmware-setup)
  - [Blynk Setup](#blynk-setup)
  - [Discord Webhook Setup](#discord-webhook-setup)
  - [Configuration](#configuration)
  - [Flashing the Firmware](#flashing-the-firmware)
- [How to Use](#how-to-use)
  - [LED Status Indicators](#led-status-indicators)
  - [Blynk Dashboard](#blynk-dashboard)
  - [Discord Notifications](#discord-notifications)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

AirCast is a air quality monitoring system designed for indoor and outdoor environments. It continuously reads temperature, humidity, atmospheric pressure, CO₂/TVOC levels, and particulate matter, then streams all data to a Blynk dashboard and sends threshold-based alerts to a Discord channel via webhook.

Built around the ESP32, AirCast is designed to be self-contained, easy to deploy, and simple to extend.

---

## Hardware

| Component | Details |
|-----------|---------|
| MCU | Onboard ESP32 Module  |
| Power | 3v3, 5v 2.54mm Pins  |
| Connectivity | Wi-Fi 802.11 b/g/n, Ethernet, Bluetooth |
| PCB | 1.6mm FR-4 2-layer 94.3 * 99.4mm |
| 4 Status LEDs | Active-low |

---

## Sensors

| Sensor | Measurement | Interface |
|--------|-------------|-----------|
| **SHT40** | Temperature &  Humidity | I2C |
| **BMP280** | Atmospheric Pressure & Temperature | I2C |
| **CCS811** | CO₂ (eCO₂) & TVOC | I2C |
| **PMS5003** | PM1.0 / PM2.5 / PM10  | UART |

---

## Features

- **Real-time Blynk dashboard** — live graphs, gauges, and historical data
- **Discord webhook notifications** — instant alerts when readings exceed configurable thresholds
- **LED status indicators** — visual feedback for power, Wi-Fi, and sensor status (active-low)
- **Configurable thresholds** — set your own alert levels for each parameter
- **Auto-reconnect** — recovers from Wi-Fi drops automatically
- **Dual temperature source** — cross-validates SHT40 and BMP280 readings

---

## Getting Started

### Prerequisites

**Software**
- [PlatformIO](https://platformio.org/) or Arduino IDE 2.x (recommended)
- Python 3.x (for PlatformIO)
- Git

**Libraries** (auto-installed via `platformio.ini` or install manually in Arduino IDE)

| Library | Purpose |
|---------|---------|
| `Blynk` | Cloud dashboard |
| `Adafruit SHT4x` | SHT40 driver |
| `Adafruit BMP280` | BMP280 driver |
| `Adafruit CCS811` | CCS811 driver |
| `ArduinoJson` | Discord webhook payload |

> **Note:** PMS5003 is handled by a custom UART parser — no additional library needed.

**Accounts**
- [Blynk](https://blynk.io) account (free tier works)
- Discord server with permission to create webhooks

---

### Firmware Setup

```bash
git clone https://github.com/yourusername/aircast.git
cd aircast
```

If using PlatformIO, dependencies will resolve automatically on first build.

If using Arduino IDE, install the libraries listed above via the Library Manager.

---

### Blynk Setup

1. Create a new **Template** in Blynk Console.
2. Add the following **Datastreams** (Virtual Pins):

| Virtual Pin | Name | Type | Unit |
|-------------|------|------|------|
| V0 | Temperature (SHT40) | Double | °C |
| V1 | Humidity | Double | % |
| V2 | Pressure | Double | hPa |
| V3 | eCO₂ | Integer | ppm |
| V4 | TVOC | Integer | ppb |
| V5 | PM1.0 | Integer | µg/m³ |
| V6 | PM2.5 | Integer | µg/m³ |
| V7 | PM10 | Integer | µg/m³ |
| V8 | Temperature (BMP280) | Double | °C |

3. Create a **Device** from the template and note down your:
   - `BLYNK_TEMPLATE_ID`
   - `BLYNK_TEMPLATE_NAME`
   - `BLYNK_AUTH_TOKEN`

4. Build a dashboard using gauges and chart widgets mapped to the virtual pins above.

---

### Discord Webhook Setup

1. In your Discord server, go to **Server Settings → Integrations → Webhooks**.
2. Click **New Webhook**, name it (e.g., `AirCast Alerts`), choose a channel, and click **Copy Webhook URL**.
3. Paste the URL into `config.h` as `DISCORD_WEBHOOK_URL`.

---

### Configuration

Copy the example config and fill in your credentials:

```bash
cp src/config.example.h src/config.h
```

Edit `src/config.h`:

```cpp
// Wi-Fi
#define WIFI_SSID        "your_wifi_ssid"
#define WIFI_PASSWORD    "your_wifi_password"

// Blynk
#define BLYNK_TEMPLATE_ID   "TMPLxxxxxx"
#define BLYNK_TEMPLATE_NAME "AirCast"
#define BLYNK_AUTH_TOKEN    "your_blynk_token"

// Discord
#define DISCORD_WEBHOOK_URL "https://discord.com/api/webhooks/xxx/yyy"

// I2C Pins
#define I2C_SDA  15
#define I2C_SCL  16

// PMS5003 UART Pins
#define PMS_RX   16
#define PMS_TX   17

// Alert Thresholds
#define THRESHOLD_CO2      1000   // ppm
#define THRESHOLD_TVOC     500    // ppb
#define THRESHOLD_PM25     35     // µg/m³
#define THRESHOLD_PM10     150    // µg/m³
#define THRESHOLD_HUMIDITY 80     // %
```

---

### Flashing the Firmware

**PlatformIO (recommended)**

```bash
pio run --target upload
pio device monitor   # view serial output at 115200 baud
```

**Arduino IDE**

1. Select board: `ESP32 Dev Module`
2. Set Upload Speed: `115200`
3. Select the correct COM port (Usually at COM6 or COM7 depending on Computer)
4. Click **Upload**

---

## How to Use

### LED Status Indicators

All LEDs are **active-low** — they light up when the pin is driven LOW.

| LED | Color | State | Meaning |
|-----|-------|-------|---------|
| PWR | Blue | Solid ON | Device powered |
| WiFi | 1 Green | Blinking | Connecting to Wi-Fi |
| WiFi | 1 Green | Solid ON | Wi-Fi connected |
| Sensor | 2 Green | Blinking | Sensor init in progress |
| Sensor | 2 Green | Solid ON | All sensors OK |
| Alert | Red | Blinking | One or more thresholds exceeded |
| Error | Red | Solid ON | Sensor failure / fatal error |

---

### Blynk Dashboard

Once the device is online and connected to Blynk, open the **Blynk app** (iOS/Android) or the **Blynk Web Console** and navigate to your device.

- Gauges update every **30 seconds** by default (configurable in `config.h` via `SEND_INTERVAL_MS`)
- Historical charts retain data for up to 1 month on the free Blynk tier
- Use Blynk's built-in notification system for mobile push alerts (optional, separate from Discord)

---

### Discord Notifications

AirCast sends a formatted embed to your Discord channel when any reading exceeds its configured threshold. Alerts are **rate-limited** to once per 5 minutes per parameter to prevent spam.

Example alert message:

```
AirCast Alert — PM2.5 High
PM2.5: 47 µg/m³  (threshold: 35 µg/m³)
Location: [device name]
Time: 2025-06-22 14:32:10
```

A **clearance message** is sent automatically when readings return to normal.

---

## Troubleshooting

### Device won't connect to Wi-Fi

- Double-check `WIFI_SSID` and `WIFI_PASSWORD` in `config.h` — both are case-sensitive.
- Ensure your network is 2.4 GHz (ESP32 doesn't support 5 GHz).
- Watch the serial monitor at 115200 baud for connection status messages.
- Try moving the device closer to the router during initial setup.

---

### Blynk shows "Device Offline"

- Confirm `BLYNK_AUTH_TOKEN` is correct and matches the device (not the template).
- Check that the ESP32 is actually connected to Wi-Fi (WiFi LED solid).
- Verify Blynk server is reachable — some networks block outbound connections on port 80/443.
- Try a full power cycle (unplug and replug).

---

### CCS811 not initializing

CCS811 requires a **48-hour burn-in** period for accurate readings. On first boot it may report `0 ppm eCO₂` — this is normal.

- Ensure `WAKE` pin on CCS811 is pulled LOW.
- CCS811 needs temperature and humidity compensation from SHT40. If SHT40 fails first, CCS811 readings will be less accurate.
- If the sensor LED stays blinking after 10 seconds, check I2C address (default `0x5A`; jumper changes it to `0x5B`).

---

### PMS5003 reads all zeros

- Check that `PMS_RX` and `PMS_TX` are correctly mapped in `config.h` — TX of sensor goes to RX of ESP32 and vice versa.
- PMS5003 takes about **30 seconds** to stabilize after power-on. Give it time before flagging a failure.
- Confirm the sensor fan is spinning (audible). No fan spin = power issue on the 5V rail.
- Verify the 5V supply to PMS5003 is stable — it draws up to 100 mA at peak and a weak supply will cause intermittent zero reads.

---

### BMP280 / SHT40 reading wrong temperature

- Both sensors are affected by PCB self-heating. Mount AirCast in a well-ventilated enclosure.
- BMP280 temperature is primarily a compensation value for pressure — use SHT40 as the primary temperature source.
- Allow 5–10 minutes warm-up time after power-on for stable readings.

---

### Discord webhook not sending

- Test the webhook URL directly with a tool like `curl` or Postman before blaming the firmware.
- Confirm the `DISCORD_WEBHOOK_URL` in `config.h` is the full URL (starts with `https://discord.com/api/webhooks/`).
- Discord rate-limits webhooks to 30 requests per minute per URL. AirCast's built-in cooldown should stay well within this, but check if multiple devices share one webhook URL.
- Watch serial output for HTTP response codes — `204` is success, `429` means rate-limited.

---

### I2C bus lockup (sensors stop responding mid-operation)

- Power cycle the device.
- Add pull-up resistors on SDA/SCL if not already present on the PCB (typically 4.7 kΩ to 3.3 V).
- Reduce I2C clock speed in firmware to 100 kHz if running at 400 kHz.
- Check that no two I2C devices share the same address — CCS811 address conflict is a common cause.

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you'd like to change.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add some feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

Please keep code style consistent and document any new config options in `config.example.h`.

---

## License

This project is licensed under the **MIT License** — see [LICENSE](LICENSE) for details.

---
*Built with ❤️ and a soldering iron.*

*Some of this page has beeen genarated by AI.*
