# BrAiNPlug

[![ESPHome](https://img.shields.io/badge/ESPHome-compatible-blue.svg)](https://esphome.io)
[![Platform](https://img.shields.io/badge/platform-ESP8266-orange.svg)](https://www.espressif.com/en/products/socs/esp8266)
[![License](https://img.shields.io/badge/license-BrAiNPub_OSO_FFA-green.svg)](https://bk-net.tk)

ESPHome based smart plug firmware for ESP8266.

BrAiNPlug adds advanced timer handling, power recovery behavior and persistent runtime tracking to compatible ESP8266 smart plugs.

The firmware is designed to work with Home Assistant through ESPHome and provides a local web interface for configuration, monitoring and control.

---

# Features

* ESPHome based firmware
* ESP8266 / ESP8285 support
* Persistent relay state handling
* Power recovery modes
* Daily ON/OFF timer scheduling with Auto and Single modes
* Weekly schedule with WeeklyAuto and WeeklySingle modes
* Web based Weekly Schedule Builder (grid painter, generates the schedule string)
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

BrAiNPlug uses a modular ESPHome configuration.

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
git clone https://github.com/BrAiNeeBug/BrAiNPlug.git

cd BrAiNPlug
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

BrAiNPlug supports optional power measurement using the HLW8012 / BL0937 energy monitoring chip.

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

The `TimerMode` selector offers five modes: `OFF`, `Auto`, `Single`, `WeeklyAuto` and `WeeklySingle`.

## OFF

Timer functionality disabled.

---

## Auto

The firmware continuously checks the current time and enforces the correct relay state, using the daily `TurnOnTime` / `TurnOffTime` fields.

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

The relay switches exactly at the configured ON and OFF times (`TurnOnTime` / `TurnOffTime`).

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

## Weekly Schedule (WeeklyAuto / WeeklySingle)

Both weekly modes read the same `WeeklySchedule` text field, which holds one or more entries:

```
HHMM-HHMM:Days;HHMM-HHMM:Days;...
```

* `HHMM-HHMM` is the start and end time of a slot.
* `Days` is any combination of the two-letter tokens `Su Mo Tu We Th Fr Sa`, no separator needed.

Example:

```
0600-0800:MoTuWeThFr;1800-2200:SaSu;1200-1230:We
```

The easiest way to build this string is the web based **[Weekly Schedule Builder](wsb.html)**: paint the desired ON times on a day/hour grid and it generates the schedule string for you (paste it into the `WeeklySchedule` field, or send it directly to the plug's IP).

The `TimerMode` dropdown decides *how* the schedule is applied - the text field itself doesn't need to change between the two modes:

### WeeklyAuto

The relay is held ON continuously for the whole duration of every matching slot, and forced OFF outside of them - the weekly equivalent of `Auto`. Survives reboots during an active slot, just like `Auto` does.

### WeeklySingle

The relay only pulses ON once at a slot's start time and OFF once at its end time - the weekly equivalent of `Single`. Between pulses the timer doesn't touch the relay, so it can be freely toggled by hand (button, web UI, Home Assistant) without being fought over.

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

Shows the calculated daily ON duration (`Auto` / `Single` modes).

Example:

```
16.0h
```

---

## TimerOFFDuration

Shows the calculated daily OFF duration (`Auto` / `Single` modes).

Example:

```
8.0h
```

---

## WeeklyStatus

Shows how many entries of the `WeeklySchedule` match the current weekday, and the resulting relay state (`WeeklyAuto` / `WeeklySingle` modes).

Example:

```
2 entries today, relay ON
```

---

## WeeklyScheduleBuilder

Shows the link to the web based Weekly Schedule Builder tool used to paint and generate the `WeeklySchedule` string.

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
* Correct ON/OFF times (`Auto` / `Single`) or a valid `WeeklySchedule` string (`WeeklyAuto` / `WeeklySingle`)

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
