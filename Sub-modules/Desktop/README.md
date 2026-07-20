# Desktop Sub-module

The **Desktop** sub-module is an environment-sensing and time-keeping expansion board for the **ESP-HI-PLUS**. It transforms the voice assistant into a desktop companion capable of monitoring room conditions and detecting human presence.

![Desktop 3D Model](Desktop%203D%20Model%20Image.png)

## Features

### Core Compute

- **RP2040** (U3) — Dual-core ARM Cortex-M0+ microcontroller to handle local sensor processing and timing.
- **16 Mbit Flash** (U1) — Storage for RP2040 firmware.

### Environment Sensors

- **SHT40** (U9) — High-accuracy relative humidity and temperature sensor.
- **STHS34PF80** (U6) — Infrared sensor for presence and motion detection.
- **VEML7700** (U7) — Ambient light sensor for automatic display brightness adjustment.

### Time Keeping

- **MCP79410** (U8) — I²C Real-Time Clock with SRAM and battery backup.
- **CR2032 Battery** (BT1) — Ensures the clock keeps time even when the main board is powered off.

### Identification & Metadata

- **R_ID (383 kΩ)** — Identifies the module as **Class 3 (Sensor — Environmental)**.
- **AT24CS02** (U10) — 2 Kbit I²C EEPROM carrying module metadata and serial number.

### User Interface & IO

- **SK9822-EC20** (D2) — Addressable RGB LED for status or mood lighting.
- **Tactile Switches** (SW1, SW3) — General purpose user inputs.
- **USB-C Port** (J1) — For independent programming of the RP2040.

## Integration

This module identifies itself as an environmental sensor class. The ESP-HI-PLUS main board can communicate with the onboard sensors via I²C and with the RP2040 via UART through the magnetic pogo-pin connectors.

### Pinouts (Magnetic Pogo Connectors)

The Desktop module uses two 5-pin magnetic pogo-pin connectors to interface with the main board.

#### J2 (MAG1)

| Pin | Name   | Description                                           |
|-----|--------|-------------------------------------------------------|
| 1   | VMAIN  | 2-way power bus shared with main module               |
| 2   | GND    | Ground                                                |
| 3   | UART RX| From main module for data transmission                |
| 4   | UART TX| From main module for data transmission                |
| 5   | EN     | Main module control to enable/disable submodule power |

#### J3 (MAG2)

| Pin | Name     | Description                                     |
|-----|----------|-------------------------------------------------|
| 1   | I2C SDA  | For reading module ID metadata EEPROM           |
| 2   | I2C SCL  | For reading module ID metadata EEPROM           |
| 3   | AUX1/ID0 | Hardware class identifier resistor              |
| 4   | AUX2     | Auxiliary data pin (unused)                     |
| 5   | AUX3     | Auxiliary data pin (unused)                     |

## Repository Layout

```text
.
├── Desktop.kicad_pro           # KiCad project file
├── Desktop.kicad_sch           # Schematic
├── Desktop.kicad_pcb           # PCB layout
├── Desktop Symbols/            # Project-local schematic symbols
├── Desktop Footprints/         # Project-local footprint libraries
├── Desktop 3D Models/          # STEP models
└── production/                 # Manufacturing outputs (BOM, Gerbers)
```
