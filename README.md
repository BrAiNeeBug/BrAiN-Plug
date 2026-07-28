# BrAiN Plug
[![ESPHome](https://img.shields.io/badge/ESPHome-compatible-blue.svg)](https://esphome.io)
[![Platform](https://img.shields.io/badge/platform-ESP8266-orange.svg)](https://www.espressif.com/en/products/socs/esp8266)
[![License](https://img.shields.io/badge/license-BrAiNPub_OSO_FFA-green.svg)](https://bk-net.tk)

ESPHome based smart plug firmware for ESP8266.

BrAiN Plug adds advanced timer handling, power recovery behavior and persistent runtime tracking to compatible ESP8266 smart plugs.

The firmware is designed to work with Home Assistant through ESPHome and provides a local web interface for configuration, monitoring and control.

---

# Features

* ESPHome based firmware
* ESP8266 / ESP8285 support
* Persistent relay state handling
* Power recovery modes
* Daily ON/OFF timer scheduling with Auto and Single modes
* Timer enable/disable
* Runtime counter with power-loss recovery
* Optional HLW8012 / BL0937 power monitoring
* Local ESPHome web interface
* OTA firmware updates
* No MQTT required

---

# Supported Devices

Currently tested with:

* S-20
* BSD-33
* EU3S

Adding support for additional ESP8266 based smart plugs is easy by creating a device specific configuration.

---

# Hardware

BrAiN Plug uses a modular ESPHome configuration.

The device specific configuration contains the hardware pin definitions and loads the common firmware packages.

## Example configuration (BSD-33)

```yaml
substitutions:
  device_name: bsd331
  friendly_name: bsd331
  board_type: esp8285

  relay_pin: GPIO14
  led_pin: GPIO13
  button_pin: GPIO3

  sel_pin: GPIO12
  cf_pin: GPIO4
  cf1_pin: GPIO5

  voltage_divider: "1534"
  current_resistor: "0.000994"

packages:
  base: !include base_brainplug.yaml
  power: !include pwrmeter_brainplug.yaml

api:
  encryption:
    key: !secret bsd331__encryption_key
  reboot_timeout: 0s
```

> GPIO assignments depend on your hardware.
> Always verify the pinout before flashing.

---

# Installation

## Requirements

You need:

* ESPHome installed
* USB connection for the first flash
* ESP8266 compatible smart plug

Install ESPHome:

```bash
pip install esphome
```

Alternatively, you can use the ESPHome Add-on inside Home Assistant.

---

# First Flash

Connect the ESP8266 device via USB.

Clone the repository:

```bash
git clone https://github.com/BrAiNeeBug/BrAiN-Plug.git

cd BrAiN-Plug
```

Create your `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI"
wifi_password: "YOUR_PASSWORD"

ota_password: "YOUR_OTA_PASSWORD"

ap_password: "YOUR_FALLBACK_PASSWORD"

webgui_password: "YOUR_WEB_PASSWORD"
```

Compile the firmware:

```bash
esphome compile base_brainplug.yaml
```

Flash via USB:

```bash
esphome run base_brainplug.yaml
```

After the first installation, firmware updates can be performed over OTA.

---

# OTA Update

When the device is connected to WiFi:

```bash
esphome run base_brainplug.yaml
```

ESPHome automatically detects the device and uploads the new firmware.

---

# Power Meter Configuration

BrAiN Plug supports optional power measurement using the HLW8012 / BL0937 energy monitoring chip.

The power meter functionality is separated into an additional ESPHome package:

```
pwrmeter_brainplug.yaml
```

To enable power measurement, include the package in your device configuration:

```yaml
packages:
  base: !include base_brainplug.yaml
  power: !include pwrmeter_brainplug.yaml
```

The required GPIO pins and calibration values must be defined in the device configuration.

Example:

```yaml
substitutions:
  sel_pin: GPIO12
  cf_pin: GPIO4
  cf1_pin: GPIO5

  voltage_divider: "1534"
  current_resistor: "0.000994"
```

> Calibration values depend on the hardware design of the smart plug.
> Different models may require different values.

---

## Available Power Sensors

When enabled, the following sensors are available:

### Voltage

Shows the current mains voltage.

Example:

```
230.5 V
```

---

### Current

Shows the current load current.

Example:

```
3.15 A
```

---

### Power

Shows the current power consumption.

Example:

```
725 W
```

---

### TotalEnergy

Shows accumulated energy consumption.

Example:

```
12.45 kWh
```

---

## Power Meter Configuration Example

`pwrmeter_brainplug.yaml`

```yaml
sensor:
  - platform: hlw8012
    model: bl0937

    sel_pin:
      number: ${sel_pin}
      inverted: true

    cf_pin: ${cf_pin}
    cf1_pin: ${cf1_pin}

    voltage_divider: ${voltage_divider}
    current_resistor: ${current_resistor}

    voltage:
      name: Voltage(V)
      web_server:
        sorting_weight: 10

    current:
      name: Current(A)
      web_server:
        sorting_weight: 11

    power:
      name: Power(W)
      web_server:
        sorting_weight: 12

    energy:
      name: TotalEnergy
      web_server:
        sorting_weight: 13

    update_interval: 10s
```

---

## Power Meter Calibration

The HLW8012 / BL0937 calibration values should be adjusted using a reliable multimeter or power meter.

Typical calibration procedure:

1. Connect a known load.
2. Compare displayed voltage/current/power values.
3. Adjust:

   * `voltage_divider` for voltage accuracy
   * `current_resistor` for current accuracy
4. Repeat until the values match.

Example:

```yaml
voltage_divider: "2050"
current_resistor: "0.00121"
```

> Incorrect calibration can result in incorrect power and energy measurements.

---

# Configuration

## Power Mode

Controls the relay state after reboot.

### Last State

Restores the previous relay state.

### Always ON

Relay turns on after boot.

### Always OFF

Relay turns off after boot.

---

# Timer Modes

## OFF

Timer functionality disabled.

---

## Auto

The firmware continuously checks the current time and enforces the correct relay state.

Example:

```
ON   06:00
OFF  22:00
```

After reboot:

```
06:00 - 22:00 = Relay ON

22:00 - 06:00 = Relay OFF
```

This mode is useful when the device loses power during an active timer period.

---

## Single

The relay switches exactly at the configured ON and OFF times.

Example:

```
ON   06:00
OFF  22:00
```

Result:

```
06:00 -> Relay ON
22:00 -> Relay OFF
```

---

# Runtime Tracking

The sensor:

```
ActualDuration
```

shows how long the relay has been in the current state.

The timestamp is stored permanently:

```yaml
last_switch_time:
  restore_value: true
```

After a reboot the runtime information is restored when RTC memory is available.

After a complete power loss, runtime tracking starts again after the next relay state change.

The firmware immediately saves relay changes:

```cpp
global_preferences->sync();
```

This prevents losing the last relay change during sudden power loss.

---

# Sensors

## SystemClock

Displays the current ESP time.

Example:

```
18:25:10
```

---

## ActualDuration

Shows the current relay runtime.

Example:

```
2h 15m
```

---

## TimerONDuration

Shows the calculated daily ON duration.

Example:

```
16.0h
```

---

## TimerOFFDuration

Shows the calculated daily OFF duration.

Example:

```
8.0h
```

---

# Web Interface

The integrated ESPHome web server provides:

* Relay control
* Timer configuration
* Power mode selection
* Runtime information
* Restart button
* Logger settings

Access:

```
http://DEVICE_IP
```

---

# Flash Wear Protection

The firmware uses ESPHome preference storage.

Recommended configuration:

```yaml
preferences:
  flash_write_interval: 30s
```

Normal ESPHome preference writes remain delayed.

The important runtime timestamp is stored immediately after relay changes to prevent losing runtime information after unexpected power loss.

---

# Troubleshooting

## Device does not boot

Check GPIO0.

GPIO0 is also the ESP8266 boot mode pin.

Incorrect wiring can prevent normal startup.

---

## Timer does not switch

Check:

* WiFi connection
* SNTP time synchronization
* TimerMode is not set to OFF
* Correct ON/OFF times

---

## ActualDuration shows NA

The device needs a valid time source.

Wait until:

```
SystemClock
```

shows a valid time.

---

# Project Information

Platform:

```
ESPHome
ESP8266 / ESP8285
```

---

# Credits

Built with:

* ESPHome
* Home Assistant
* ESP8266 platform

---

# License

```
BrAiNPub_OSO_FFA

(c)2026 by BrAiNee
```
