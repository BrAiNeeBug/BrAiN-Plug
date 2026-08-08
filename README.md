# BrAiNPlug (Beta-Status, use with care!)

[![ESPHome](https://img.shields.io/badge/ESPHome-compatible-blue.svg)](https://esphome.io)
[![Platform](https://img.shields.io/badge/platform-ESP8266%20%2F%20ESP32-orange.svg)](https://www.espressif.com/)
[![License](https://img.shields.io/badge/license-BrAiNPub_OSO_FFA-green.svg)](https://bk-net.tk)

ESPHome based smart plug firmware for **ESP8266 / ESP8285 and ESP32**.

BrAiNPlug adds advanced timer handling, power recovery behavior, persistent runtime tracking and optional power monitoring to compatible smart plugs.

The firmware is designed to work with Home Assistant through ESPHome and provides a local web interface.

**🔗 [Weekly Schedule Builder](https://braineebug.github.io/BrAiNPlug/wsb.html)**  
Paint your weekly ON/OFF schedule on a grid and generate the required `TimerConf` string.

---

# Features

- ESPHome based firmware
- ESP8266 / ESP8285 / ESP32 support
- Low-Power Consumption your Plug stays C☻☻L
- Persistent relay state handling
- Power recovery modes
- Daily timer modes
- Weekly timer modes
- Duration timer with ON/OFF cycle times
- One single `TimerConf` field for all timer configurations
- Web based Weekly Schedule Builder
- Timer enable/disable
- ChildLock
- Runtime counter with power-loss handling
- Optional HLW8012 / BL0937 power monitoring
- Persistent `TotalEnergy` tracking
- `ResetTotalEnergy` button
- Local ESPHome web interface
- OTA firmware updates
- No MQTT required

---

# Repository Structure

The firmware is split into reusable ESPHome packages:

```text
BrAiNPlug/
├── README.md
├── base-brainplug.yaml
├── base-brainplug-esp32.yaml
├── pwrmeter-brainplug.yaml
├── brainplug-configs.nfo
└── wsb.html
```

### `base-brainplug.yaml`

Main firmware package for **ESP8266 / ESP8285** devices.

### `base-brainplug-esp32.yaml`

Main firmware package for **ESP32** devices.

### `pwrmeter-brainplug.yaml`

Optional power-meter package for devices using an **HLW8012 / BL0937** compatible measurement circuit.

### `brainplug-configs.nfo`

Contains the device-specific configurations and pin/calibration values for supported plugs.

### `wsb.html`

Web based Weekly Schedule Builder.

---

# Supported Devices

Currently configured/tested devices include:

- S-20
- BSD-33
- GoKlug EU3S
- NOUS A8T

The NOUS A8T configuration uses the ESP32 base package.

Additional ESPHome-based smart plugs can be supported by creating a device-specific configuration with the correct GPIO assignments and, if applicable, power-meter calibration values.

**Always verify the hardware pinout before flashing.**

---

# Hardware Configuration

BrAiNPlug uses a modular ESPHome configuration.

A device configuration defines the hardware-specific substitutions and includes the appropriate base package.

The current repository uses two different base packages:

```yaml
# ESP8266 / ESP8285
packages:
  base: !include base-brainplug.yaml
```

or:

```yaml
# ESP32
packages:
  base: !include base-brainplug-esp32.yaml
```

If the device has a power meter:

```yaml
packages:
  base: !include base-brainplug.yaml
  power: !include pwrmeter-brainplug.yaml
```

For ESP32:

```yaml
packages:
  base: !include base-brainplug-esp32.yaml
  power: !include pwrmeter-brainplug.yaml
```

---

# Example: ESP8266 / ESP8285

Example based on the BSD-33 configuration:

```yaml
substitutions:
  device_name: bsd33
  friendly_name: bsd33
  board_type: esp8285

  relay_pin: GPIO14
  led_pin: GPIO13
  button_pin: GPIO3

  sel_pin: GPIO12
  cf_pin: GPIO4
  cf1_pin: GPIO5

  voltage_divider: "1534"
  current_resistor: "0.000994"
  power_multiply: "1.288"

  localdomain: ".local"

packages:
  base: !include base-brainplug.yaml
  power: !include pwrmeter-brainplug.yaml

api:
  encryption:
    key: !secret bsd33__encryption_key
  reboot_timeout: 0s
```

---

# Example: ESP32

Example based on the NOUS A8T configuration:

```yaml
substitutions:
  device_name: nousa8t1
  friendly_name: nousa8t1
  esp32_board: esp32dev

  relay_pin: GPIO13
  led_pin: GPIO02
  button_pin: GPIO04

  sel_pin: GPIO14
  cf_pin: GPIO27
  cf1_pin: GPIO26

  voltage_divider: "1514"
  current_resistor: "0.001271"
  power_multiply: "1.310"

  localdomain: ".local"

packages:
  base: !include base-brainplug-esp32.yaml
  power: !include pwrmeter-brainplug.yaml

api:
  encryption:
    key: !secret nous_a8t__encryption_key
  reboot_timeout: 0s
```

The exact GPIO assignments and calibration values depend on the hardware.

---

# Important Substitutions

The following values are normally device-specific:

```yaml
device_name:
friendly_name:
board_type:       # ESP8266 / ESP8285
esp32_board:      # ESP32
relay_pin:
led_pin:
button_pin:
localdomain:
```

For power-meter devices additionally:

```yaml
sel_pin:
cf_pin:
cf1_pin:
voltage_divider:
current_resistor:
power_multiply:
```

`localdomain` needs the leading dot:

```yaml
localdomain: ".local"
```

or for another local DNS suffix:

```yaml
localdomain: ".bk-net"
```

---

# Installation

## Requirements

You need:

- ESPHome installed, or the ESPHome Add-on inside Home Assistant
- A compatible ESP8266 / ESP8285 / ESP32 smart plug
- USB/serial access for the first flash when the device does not already have an OTA-capable firmware

Clone the repository:

```bash
git clone https://github.com/BrAiNeeBug/BrAiNPlug.git
cd BrAiNPlug
```

Create or update your `secrets.yaml`:

```yaml
wifi_ssid: "YOUR_WIFI"
wifi_password: "YOUR_PASSWORD"

ota_password: "YOUR_OTA_PASSWORD"

ap_password: "YOUR_FALLBACK_PASSWORD"

webgui_password: "YOUR_WEB_PASSWORD"
```

---

# Creating a Device Configuration

The device-specific YAML contains:

1. Hardware substitutions
2. The correct BrAiNPlug base package
3. The optional power-meter package
4. The ESPHome API configuration

Ready-made configurations are available in:

```text
brainplug-configs.nfo
```

Copy the matching configuration into a new YAML file and adjust the values for your hardware.

---

# Home Assistant ESPHome Add-on

This is the recommended way for Home Assistant users.

1. Open the **ESPHome** dashboard.
2. Create a new device.
3. Select the correct ESP8266 or ESP32 board.
4. Open the generated YAML.
5. Replace it with the appropriate BrAiNPlug device configuration.
6. Make sure the required secrets are available.
7. Compile and install the firmware.

If the device already runs ESPHome or another OTA-capable firmware, the firmware can normally be installed wirelessly.

---

# ESPHome CLI

For command-line installations:

```bash
esphome compile bsd33.yaml
esphome run bsd33.yaml
```

Replace `bsd33.yaml` with your actual device configuration.

---

# Flashing from Tasmota

If the smart plug already runs Tasmota and supports OTA firmware upload, a serial connection is normally not required for ESP82x!
On ESP32 you HAVE to use a serial connection with Factory-Image otherwise you will brik the Plug and following OTA-Updates!

Compile the BrAiNPlug firmware:

```bash
esphome compile bsd33.yaml
```

Then upload the generated `.bin` file through the Tasmota web interface:

```text
Firmware Upgrade -> Upload
```

The exact procedure depends on the Tasmota version and device.

---

# Power Meter

BrAiNPlug supports optional power measurement using an HLW8012 / BL0937 compatible measurement circuit.

Power monitoring is separated into:

```text
pwrmeter-brainplug.yaml
```

Enable it by adding:

```yaml
packages:
  base: !include base-brainplug.yaml
  power: !include pwrmeter-brainplug.yaml
```

For ESP32:

```yaml
packages:
  base: !include base-brainplug-esp32.yaml
  power: !include pwrmeter-brainplug.yaml
```

The required GPIO pins and calibration values must be defined by the device configuration.

---

# Power Meter Calibration

The following values are hardware-specific:

```yaml
voltage_divider: "2050"
current_resistor: "0.00121"
power_multiply: "1.0"
```

Typical calibration procedure:

1. Connect a known load.
2. Compare the displayed voltage/current/power with a reliable reference meter.
3. Adjust `voltage_divider` for voltage accuracy.
4. Adjust `current_resistor` for current accuracy.
5. Adjust `power_multiply` for final power fine-tuning.
6. Repeat until the readings match.

`power_multiply` is an additional linear correction factor.

For example:

```yaml
power_multiply: "1.05"
```

adds approximately 5% to the calculated power value.

Incorrect calibration can result in incorrect power and energy measurements.

---

# Available Power Sensors

When the power-meter package is enabled:

### Voltage

Displays the current mains voltage.

```text
230.5 V
```

### Current

Displays the current load.

```text
3.15 A
```

### Power

Displays the current power consumption.

```text
725 W
```

### TotalEnergy

Displays accumulated energy consumption in kWh.

```text
12.450 kWh
```

`TotalEnergy` is stored persistently.

A `ResetTotalEnergy` button is available to reset the accumulated value.

---

# Configuration

## PowerONMode

Controls the relay state after boot/reboot.

### Last State

Restores the previously stored relay state.

### Always ON

Turns the relay on after boot.

### Always OFF

Turns the relay off after boot.

---

# ChildLock

`ChildLock` blocks manual relay switching while enabled.

When `ChildLock` is ON:

- The physical button is ignored.
- The Home Assistant switch cannot manually change the relay.
- The web interface switch cannot manually change the relay.

The timer is **not** blocked by ChildLock.

Power-recovery behavior is also not blocked.

This makes ChildLock useful when the timer should have exclusive control over the relay.

---

# Timer System

BrAiNPlug now uses **one single configuration field**:

```text
TimerConf
```

The selected `TimerMode` determines how this field is interpreted.

There are six available modes:

```text
OFF
Auto
Single
WeeklyAuto
WeeklySingle
Duration
```

The timer configuration is therefore no longer split into separate `TurnOnTime`, `TurnOffTime` and `WeeklySchedule` fields.

---

# TimerConf

`TimerConf` is one text field.

The required format depends on the selected `TimerMode`.

| TimerMode | TimerConf format |
|---|---|
| `OFF` | ignored |
| `Auto` | `HH:MM:SS-HH:MM:SS` |
| `Single` | `HH:MM:SS-HH:MM:SS` |
| `WeeklyAuto` | `HHMM-HHMM:Days;...` |
| `WeeklySingle` | `HHMM-HHMM:Days;...` |
| `Duration` | `HH:MM:SS/HH:MM:SS` |

---

# OFF

Timer functionality is disabled.

The relay can be controlled manually through the button, Home Assistant or the web interface, subject to ChildLock.

---

# Auto

`Auto` continuously evaluates the current time and enforces the configured relay state.

Example:

```text
TimerMode: Auto
TimerConf: 06:00:00-22:00:00
```

Result:

```text
06:00:00 -> ON
22:00:00 -> OFF
```

After a reboot during the active period, the relay is brought back to the correct state.

This is useful when the device loses power while a timer period is active.

For timer-only operation, combine `Auto` with `ChildLock`.

---

# Single

`Single` switches the relay at the configured ON and OFF times.

Example:

```text
TimerMode: Single
TimerConf: 06:00:00-22:00:00
```

Result:

```text
06:00 -> ON
22:00 -> OFF
```

Unlike `Auto`, the relay is not continuously forced to the calculated state between the trigger times.

---

# WeeklyAuto

`WeeklyAuto` uses the same `TimerConf` field for a weekly schedule.

Format:

```text
HHMM-HHMM:Days;HHMM-HHMM:Days;...
```

Example:

```text
0600-0800:MoTuWeThFr;1800-2200:SaSu;1200-1230:We
```

The day tokens are:

```text
Su Mo Tu We Th Fr Sa
```

No separator is required between day tokens.

Example:

```text
0600-0800:MoTuWeThFr
```

means:

```text
Monday    06:00-08:00
Tuesday   06:00-08:00
Wednesday 06:00-08:00
Thursday  06:00-08:00
Friday    06:00-08:00
```

`WeeklyAuto` continuously enforces the ON/OFF state for matching schedule entries.

The schedule survives a reboot during an active time slot.

---

# WeeklySingle

`WeeklySingle` uses exactly the same `TimerConf` format as `WeeklyAuto`.

Example:

```text
TimerMode: WeeklySingle
TimerConf: 0600-0800:MoTuWeThFr;1800-2200:SaSu
```

The relay receives an ON trigger at the start of a matching slot and an OFF trigger at the end.

Between these trigger points the timer does not continuously enforce the relay state.

This allows manual switching between scheduled events.

---

# Duration

`Duration` uses the same single `TimerConf` field but interprets it as an ON/OFF cycle:

```text
HH:MM:SS/HH:MM:SS
```

The first value is the ON duration.

The second value is the OFF duration.

Example:

```text
TimerMode: Duration
TimerConf: 00:00:10/00:00:05
```

Result:

```text
10 seconds ON
5 seconds OFF
10 seconds ON
5 seconds OFF
...
```

Duration mode is evaluated with 1-second resolution.

This allows short ON/OFF cycles that are not possible with the normal daily/weekly timer modes.

---

# Weekly Schedule Builder

The easiest way to create a weekly schedule is the:

**[Weekly Schedule Builder](https://braineebug.github.io/BrAiNPlug/wsb.html)**

The builder generates a `TimerConf` string suitable for:

```text
WeeklyAuto
```

or:

```text
WeeklySingle
```

Example output:

```text
0600-0800:MoTuWeThFr;1800-2200:SaSu
```

Copy the generated string into the `TimerConf` field.

---

# Runtime Tracking

The sensor:

```text
ActualDuration
```

shows how long the relay has been in its current state.

Example:

```text
2h 15m
```

The runtime timestamp is stored persistently.

Relay state changes are immediately synchronized to preference storage to reduce the chance of losing the last change during an unexpected power loss.

The ESP8266 implementation additionally uses RTC memory for runtime recovery.

On a complete power loss, `ActualDuration` reports:

```text
PwrLoss
```

until a new valid relay-state timestamp is available.

---

# Sensors

## SystemClock

Displays the current ESP time.

Example:

```text
18:25:10
```

A valid SNTP time source is required for timer operation.

---

## ActualDuration

Displays the runtime of the current relay state.

Example:

```text
2h 15m
```

After a detected power loss:

```text
PwrLoss
```

---

## Timer0NDuration

Displays the calculated ON duration.

For `Auto` / `Single`, it is calculated from the daily `TimerConf`.

For `Duration`, it displays the configured ON duration.

Examples:

```text
16.0h
```

or:

```text
10s
```

---

## TimerOFFDuration

Displays the calculated OFF duration.

For `Auto` / `Single`, it is calculated from the daily `TimerConf`.

For `Duration`, it displays the configured OFF duration.

---

## WeeklyStatus

Displays information about matching weekly schedule entries and the resulting relay state.

Example:

```text
2 entries today, relay ON
```

---

## ScheduleBuilderURL

Provides the URL of the Weekly Schedule Builder.

It is disabled by default in the entity list.

---

# Web Interface

The integrated ESPHome web server provides:

- Relay control
- ChildLock
- TimerMode
- TimerConf
- PowerONMode
- Runtime information
- Restart
- Logger settings

Access the web interface at:

```text
http://DEVICE_IP
```

The web interface is protected by the configured `webgui_password`.

---

# Flash Wear Protection

BrAiNPlug uses ESPHome preference storage.

Recommended configuration:

```yaml
preferences:
  flash_write_interval: 30s
```

Normal preference writes remain delayed to reduce flash wear.

Important relay-state/runtime information is synchronized immediately after relay changes where required.

---

# Troubleshooting

## Device does not boot

Check the GPIO assignments.

On ESP8266 devices, special attention must be paid to boot-strapping pins such as:

```text
GPIO0
```

Incorrect wiring or an incorrect GPIO assignment can prevent normal startup.

For ESP32 devices, verify the board definition and GPIO assignments for the specific hardware.

---

## Timer does not switch

Check:

- WiFi connection
- SNTP time synchronization
- `TimerMode`
- `TimerConf`
- Correct `TimerConf` syntax for the selected mode
- ChildLock is not blocking the timer

Examples:

```text
Auto:
06:00:00-22:00:00
```

```text
WeeklyAuto:
0600-0800:MoTuWeThFr
```

```text
Duration:
00:00:10/00:00:05
```

---

## ActualDuration shows `NA`

The device needs a valid time source.

Wait until:

```text
SystemClock
```

shows a valid time.

---

## Power values are incorrect

Check the hardware-specific:

```yaml
voltage_divider:
current_resistor:
power_multiply:
```

Calibration values from another plug should not automatically be copied to a different hardware design.

---

# Project Information

Platform:

```text
ESPHome
ESP8266 / ESP8285 / ESP32
```

Main firmware packages:

```text
base-brainplug.yaml
base-brainplug-esp32.yaml
pwrmeter-brainplug.yaml
```

Project:

https://github.com/BrAiNeeBug/BrAiNPlug

Weekly Schedule Builder:

https://braineebug.github.io/BrAiNPlug/wsb.html

---

# Credits

Built with:

- ESPHome
- Home Assistant
- ESP8266 / ESP32 platforms

Special thanks:

- **Claude & ChatGPT** - CODE / README / CONFIG
- RIP Plugs: 1x EU3S (Bootloop)

---

# License

```text
BrAiNPub_OSO_FFA

(c)2026 by BrAiNee
```
