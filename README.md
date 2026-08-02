# BrAiNPlug (Beta-Status use with care!)

[![ESPHome](https://img.shields.io/badge/ESPHome-compatible-blue.svg)](https://esphome.io)
[![Platform](https://img.shields.io/badge/platform-ESP8266-orange.svg)](https://www.espressif.com/en/products/socs/esp8266)
[![License](https://img.shields.io/badge/license-BrAiNPub_OSO_FFA-green.svg)](https://bk-net.tk)

ESPHome based smart plug firmware for ESP8266.

BrAiNPlug adds advanced timer handling, power recovery behavior and persistent runtime tracking to compatible ESP8266 smart plugs.

The firmware is designed to work with Home Assistant through ESPHome and provides a local web interface.

**🔗 [Weekly Schedule Builder](https://braineebug.github.io/BrAiNPlug/wsb.html)** — paint your weekly ON/OFF schedule on a grid.

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
* ChildLock - blocks manual relay switching via the physical button and the Home Assistant / web UI switch, without affecting the timer or the power-recovery restore behavior
* Runtime counter with power-loss recovery
* Optional HLW8012 / BL0937 power monitoring
* Persistent TotalEnergy tracking with a ResetTotalEnergy button to zero the counter
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

The device specific configuration contains the hardware pin definitions, firmware substitutions and loads the common firmware packages.

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

  voltage_divider: "1516"
  current_resistor: "0.001174"
  power_multiply: "1.288"
  localdomain: ".local"

packages:
  base: !include base_brainplug.yaml
  power: !include pwrmeter_brainplug.yaml

api:
  encryption:
    key: !secret bsd331__encryption_key
  reboot_timeout: 0s
```

> GPIO assignments and calibration values depend on your hardware.
> Always verify the pinout before flashing.
> `localdomain` needs the leading dot (e.g. `.local`) - ESPHome hostnames can't contain one, so it has to be part of this value.

---

# Installation

## Requirements

You need:

* ESPHome installed (or the ESPHome Add-on inside Home Assistant)
* An ESP8266 compatible smart plug
* USB connection for the very first flash (not needed if the plug already runs Tasmota or any other OTA-capable firmware - see below)

Install ESPHome via pip:

```bash
pip install esphome
```

Then clone this repository (needed for both options below, since it contains `base_brainplug.yaml`, `pwrmeter_brainplug.yaml` and the `brainplug-configs.nfo` examples):

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

---

# Creating a Device Config & Flashing

Each device gets its own YAML configuration containing the required `substitutions`, `packages` and `api` sections.

The example above shows the minimum required configuration for BrAiNPlug. Besides the hardware pin definitions, firmware substitutions such as `localdomain` are always required; `power_multiply` (and the other power calibration values) is only needed for devices that include the power meter package - the S-20 for example doesn't need it.

Ready-made configurations for all currently supported devices are available in [`brainplug-configs.nfo`](https://github.com/BrAiNeeBug/BrAiNPlug/blob/main/brainplug-configs.nfo). Simply copy the configuration matching your device into a new YAML file (for example `bsd33.yaml`) and adjust the device-specific values as needed.

## Option A: Home Assistant ESPHome Add-on (recommended, GUI)

This is how most people will do it:

1. Open the **ESPHome** add-on/dashboard in Home Assistant.
2. Click **New Device** -> **Continue** -> pick your board (e.g. ESP8266) -> give it a name.
3. Once created, open the device's **Edit** view and replace the auto-generated YAML with your device config (copy the matching block from `brainplug-configs.nfo`, or adapt the BSD-33 example above).
4. Make sure `secrets.yaml` (in the ESPHome add-on's config folder) contains your WiFi/OTA/webgui secrets.
5. Click **Install** -> **Wirelessly** (if the device already has ESPHome/Tasmota/any OTA-capable firmware on it) or plug it in via USB the first time.

## Option B: Command line (esphome CLI)

For anyone not using Home Assistant, or scripting/CI setups:

```bash
esphome compile bsd33.yaml
esphome run bsd33.yaml
```

The first `run` needs either a USB connection or an existing OTA-capable firmware on the device; every run after that goes over OTA automatically.

> **Already running Tasmota?**
> If the plug is already flashed with Tasmota, you don't need a USB/serial connection at all. Just compile the firmware for your device (`esphome compile bsd33.yaml`), then upload the resulting `.bin` file directly through Tasmota's own web UI under **Firmware Upgrade -> Upload**. No need to open the case or solder anything.

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
  power_multiply: "1.0"
```

> Calibration values depend on the hardware design of the smart plug.
> Different models may require different values.
> `power_multiply` is an extra linear correction factor on top of the power reading (e.g. `1.05` = +5%) - use it for fine-tuning `Power(W)` against a reference meter once `voltage_divider`/`current_resistor` are already roughly calibrated. Leave it at `"1.0"` if no correction is needed.

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

Shows accumulated energy consumption, persisted across reboots.

Example:

```
12.45 kWh
```

A config-category **ResetTotalEnergy** button is available next to the sensor to zero the counter (for example at the start of a new billing period), without affecting any other stored settings.

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
        sorting_weight: 2
      icon: "mdi:sine-wave"

    current:
      name: Current(A)
      web_server:
        sorting_weight: 3
      icon: "mdi:current-ac"

    power:
      name: Power(W)
      filters:
        - multiply: ${power_multiply}
      web_server:
        sorting_weight: 4
      icon: "mdi:lightning-bolt"

    energy:
      name: TotalEnergy
      web_server:
        sorting_weight: 5
      icon: "mdi:home-lightning-bolt"

    update_interval: 5s
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
   * `power_multiply` for a final fine-tune of the power reading
4. Repeat until the values match.

Example:

```yaml
voltage_divider: "2050"
current_resistor: "0.00121"
power_multiply: "1.0"
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

## ChildLock

A config-category switch that blocks manual relay switching while active.

When **ChildLock** is ON:

* The physical button on the plug is ignored.
* The `Switch` entity in Home Assistant / the web UI can no longer turn the relay on or off - toggling it just snaps back to the actual state.

When **ChildLock** is OFF, the relay switches normally through button, Home Assistant and the web UI, same as without the feature.

Importantly, ChildLock only affects *manual* switching. It never blocks:

* The **Timer** (`Auto`, `Single`, `WeeklyAuto`, `WeeklySingle`) - schedules keep running exactly as configured.
* The **Power Mode** restore behavior on reboot / after a power loss / after an OTA update.

> **Tip:** if you're running `TimerMode: Auto` (or `WeeklyAuto`), turn **ChildLock ON** as well. That way nobody can accidentally flip the relay via the button or the app in the middle of an active ON/OFF window - the timer stays in full (100%) control of the relay, and will simply re-enforce the correct state on its next check either way.

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

> Combine this with **ChildLock ON** if you want the schedule to be the only thing controlling the relay - see the [ChildLock](#childlock) section above.

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

The easiest way to build this string is the web based **[Weekly Schedule Builder](https://braineebug.github.io/BrAiNPlug/wsb.html)**: paint the desired ON times on a day/hour grid and it generates the schedule string for you (paste it into the `WeeklySchedule` field, or send it directly to the plug's IP).

The `TimerMode` dropdown decides *how* the schedule is applied - the text field itself doesn't need to change between the two modes:

### WeeklyAuto

The relay is held ON continuously for the whole duration of every matching slot, and forced OFF outside of them - the weekly equivalent of `Auto`. Survives reboots during an active slot, just like `Auto` does.

> Just like `Auto`, combine this with **ChildLock ON** if you want the weekly schedule to be the only thing controlling the relay.

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
* ChildLock toggle
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
* `TimerMode` is set correctly (`Auto`, `Single`, `WeeklyAuto` or `WeeklySingle`)
* Correct ON/OFF times (`Auto` / `Single`) or a valid `WeeklySchedule` string (`WeeklyAuto` / `WeeklySingle`)
* **ChildLock** is unrelated to this - it never blocks the timer, only manual button/app switching

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

Special thanks:

* **Claude** & **ChatGPT** — CODE/README/CONFIG
* RIP Plugs: 1xEU3S(Bootloop)

---

# License

```
BrAiNPub_OSO_FFA

(c)2026 by BrAiNee
```
