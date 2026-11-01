# ESP8266 Relay – Water Pump Controller

A compact mains-powered ESP8266 (ESP-12E) board with a 5V relay, designed to switch a water pump (or any AC load) over Wi-Fi. Works well with ESPHome / Home Assistant.

**Board:** 2-layer PCB, ~65 × 62 mm  
**Design tool:** KiCad 10  
**Revision:** 1.0 (2026-10-16)
<img width="1152" height="728" alt="esp8266-relay-waterpump-top" src="https://github.com/user-attachments/assets/ddd7ec4c-b97a-48ff-b7ab-a9beb5aecb54" />

> ⚠️ **WARNING – MAINS VOLTAGE**  
> This board connects directly to AC mains. Only build and test it if you know how to work safely with high voltage. Always disconnect power before touching the board, use a properly rated fuse, and connect the Earth terminal. Use at your own risk.

## Features

- ESP-12E (ESP8266) Wi-Fi module
- On-board AC-DC converter (HLK-PM01, 5V) – no external adapter needed
- AMS1117-3.3 regulator for the ESP8266
- SRD-05VDC-SL-C relay (SPDT; board uses COM and NO contacts)
- Optocoupler (EL817) isolating the ESP8266 from the relay driver
- Relay status LED
- SMD fuse on the switched AC output
- Screw terminals for AC input and AC output (with Earth)
- Header for I2C sensors, UART/TTL programming header
- Reset and Flash buttons, plus a connector for an external push button
- Isolation slots in the PCB between the high-voltage and low-voltage areas

## Block Diagram

```
AC IN ──► HLK-PM01 (5V) ──► AMS1117 (3.3V) ──► ESP-12E
  │              │                                  │ GPIO13
  │              └──► Relay coil ◄── S8050 ◄── EL817 ◄┘
  │
  └──► Relay COM ──► NO ──► Fuse ──► AC OUT (pump)
```

## Pin Mapping

| ESP8266 Pin | Function |
|---|---|
| GPIO13 | Relay control (via 330R → EL817 optocoupler → S8050) |
| GPIO12 | External push button (J5, to GND) |
| GPIO4 | I2C SDA (J4, 4.7k pull-up) |
| GPIO5 | I2C SCL (J4, 4.7k pull-up) |
| GPIO0 | Flash button (SW2), 10k pull-up |
| GPIO2 | 10k pull-up |
| GPIO15 | 10k pull-down |
| GPIO1 / GPIO3 | TX / RX on TTL header (J2) |
| RST | Reset button (SW1), 10k pull-up |
| EN | 10k pull-up |

GPIO12 has no external pull-up, so enable the internal pull-up in firmware.

## Connectors

| Ref | Name | Description |
|---|---|---|
| J3 | AC IN | Mains input: L, Earth, N |
| J1 | AC OUT | Switched output: N, Earth, L (through relay and fuse) |
| J2 | TTL | 3.3V, GND, RX, TX, RST (for flashing) |
| J4 | I2C | 3.3V, GND, SCL, SDA |
| J5 | Push Button | GPIO12 and GND |

## Main Components

| Ref | Part |
|---|---|
| U1 | ESP-12E |
| U2 | EL817 optocoupler |
| U3 | HLK-PM01 (AC-DC 5V, 3W) |
| U4 | AMS1117-3.3 |
| K1 | SRD-05VDC-SL-C relay |
| Q1 | S8050 NPN transistor |
| D1 | Flyback diode (SMA) |
| D2 + R11 | Relay status LED + 330R resistor |
| F1 | SMD fuse, 2010 (5025 metric) – choose a rating that suits your load |

## Flashing

1. Connect a USB-TTL adapter (3.3V logic) to J2: GND, TX → RX, RX → TX.
2. Hold the **Flash** button (SW2), press **Reset** (SW1), then release Flash.
3. Upload your firmware (ESPHome, Arduino, etc.).

> Power the board from a USB-TTL adapter's 3.3V only while mains is **disconnected**, or power it from the mains side, but never both at once.

## Example ESPHome Config

```yaml
esp8266:
  board: esp01_1m

switch:
  - platform: gpio
    pin: GPIO13
    name: "Water Pump"
    id: water_pump

binary_sensor:
  - platform: gpio
    pin:
      number: GPIO12
      mode: INPUT_PULLUP
      inverted: true
    name: "Pump Button"
    on_press:
      - switch.toggle: water_pump

i2c:
  sda: GPIO4
  scl: GPIO5
```

## Repository Files

- `esp8266-relay-waterpump.kicad_sch` – schematic
- `esp8266-relay-waterpump.kicad_pcb` – PCB layout

Open both files with KiCad 10 or newer.

## License

MIT License
