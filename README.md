# P-NUT

this project compiles and runs nut server, currently testing waveshare.
debugging problems with USB connection to UPS - no testing on m5stack yet

**PoE Network UPS Tools Bridge**

P-NUT is an ESPHome-based firmware that bridges a USB HID UPS device to the network via the NUT (Network UPS Tools) protocol. It enables network monitoring and control of non-network-managed UPS devices using PoE-powered ESP32-P4 hardware.

## Purpose

Provides bidirectional NUT server access to a USB HID UPS over PoE Ethernet, eliminating the need for a dedicated host machine to run NUT. Exposes full UPS monitoring, control, and delay configuration via the standard NUT protocol on port 3493.

## Hardware

Two supported platforms sharing identical firmware except for board identifier:

| File | Board | Notes |
|---|---|---|
| `p-nut_waveshare.yaml` | Waveshare ESP32-P4-ETH with PoE module | Single USB OTG port — used for both flashing and UPS connection |
| `p-nut_m5stack.yaml` | M5Stack Unit PoE-P4 | Dedicated USB Host port for UPS, separate OTG port for flashing |

Both use IP101GRI Ethernet PHY with identical pin assignments (MDC=GPIO31, MDIO=GPIO52, CLK=GPIO50, RST=GPIO51).

Connect the UPS via USB-B to USB-C cable to:
- **Waveshare**: The USB OTG port (disconnect during firmware flashing)
- **M5Stack**: The USB Host port (labeled, separate from OTG/debug port)

## Dependencies

### External Components

This project uses components from:

```
github://lostsynapse/esphome-components
```

This is a personal fork of [bullshit/esphome-components](https://github.com/bullshit/esphome-components). There is one patch to resolve a crash due to an informational log entry that occurrs before logging is set up. This is otherwise functionall identical.

```yaml
external_components:
  - source: github://bullshit/esphome-components
    components: [ups_hid, nut_server]
```

Components used:
- `ups_hid` — USB HID UPS monitoring and control
- `nut_server` — NUT v1.3 compliant TCP server on port 3493

## Prerequisites

### ESPHome

ESPHome 2026.3.0 or later is required. ESP32-P4 Ethernet compilation support was fixed in 2026.3.0.

Install ESPHome in a Python virtual environment:

```bash
python3 -m venv ~/esphome-venv
source ~/esphome-venv/bin/activate
pip install esphome
```

Add to shell profile for persistent activation:

```bash
echo "source ~/esphome-venv/bin/activate" >> ~/.bashrc
```

### secrets.yaml

Create `secrets.yaml` in the project directory (this file is gitignored and must never be committed):

```yaml
api_key: "your_32_byte_base64_key_here"
ota_password: "your_ota_password_here"
```

Generate a valid API key:

```bash
python3 -c "import base64, os; print(base64.b64encode(os.urandom(32)).decode())"
```

## Network

Devices default to DHCP. To set static address, add the following to your p-nut_xxxxxx.yaml and complete for your network:

```yaml
manual_ip:
    static_ip: 192.168.x.x
    gateway: 192.168.x.x
    subnet: 255.255.255.0
```

## Compile

Compile without hardware present to validate configuration:

```bash
esphome compile p-nut_m5stack.yaml
```

```bash
esphome compile p-nut_waveshare.yaml
```

## Flash

Enter download mode on the device:
- **M5Stack Unit PoE-P4**: Hold side button for 3 seconds until green LED lights
- **Waveshare ESP32-P4-ETH**: Hold BOOT button, press RESET, release BOOT

Connect via OTG/debug USB-C port, then flash:

```bash
esphome upload p-nut_m5stack.yaml
```

```bash
esphome upload p-nut_waveshare.yaml
```

Subsequent updates can be performed OTA once the device is on the network.

## Monitor

```bash
esphome logs p-nut_m5stack.yaml
```

## NUT Client Configuration

Point NUT clients at the device IP on port 3493. Example `/etc/nut/upsmon.conf`:

```
MONITOR ups@<device-ip>:3493 1 nutuser "" slave
```

Default NUT username is `nutuser` with no password. Configure credentials in the YAML `nut_server:` block if authentication is required.

## Exposed Entities

### Sensors
- Battery Level (%)
- Input Voltage (V)
- Output Voltage (V)
- Load Percentage (%)
- Runtime Remaining (min)
- Device Uptime

### Binary Sensors
- Battery Charging
- Overload Warning

### Text Sensors
- UPS Manufacturer
- UPS Model
- UPS Test Result
- ESPHome Version

### Controls (Buttons)
- Beeper Enable / Disable / Mute / Test
- Quick Battery Test / Deep Battery Test / Stop Battery Test
- Panel Test / Stop Panel Test

### Configuration (Numbers)
- Shutdown Delay (0–600s)
- Start Delay (0–600s)
- Reboot Delay (0–600s)

## Notes

- Protocol is auto-detected from USB vendor ID. Supported: APC HID, CyberPower HID, Generic HID.
- Delay configuration write support varies by UPS protocol. APC input-only devices cannot write delays.
- The `apply_default_delays` script sets shutdown=60s, start=120s, reboot=60s and can be triggered via the ESPHome API or Home Assistant.