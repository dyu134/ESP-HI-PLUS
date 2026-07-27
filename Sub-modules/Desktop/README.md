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

The Desktop module follows the standard [ESP-HI-PLUS identification protocol](../README.md).

- **R_ID (383 kΩ)** — Identifies the module as **Class 3 (Sensor — Environmental)**.
- **AT24CS02** (U10) — 2 Kbit I²C EEPROM carrying module metadata and serial number.

#### EEPROM Metadata

| Offset | Size | Field | Value (Hex) | Value (Decoded) | Description |
| :------: | :----: | :------ | :-----------: | :---------------- | :------------ |
| 0x00 | 4 | Magic Number | 45 53 50 4D | "ESPM" | ASCII sanity check |
| 0x04 | 1 | Class ID | 0x03 | 3 | Sensor - Environmental |
| 0x05 | 1 | Module ID | 0x01 | 1 | Desktop v1 |
| 0x06 | 1 | HW Revision | 0x01 | 1 | First PCB revision |
| 0x07 | 2 | FW Revision | 0x0100 | 1.0 | Major.Minor |
| 0x09 | 1 | Protocol Revision | 0x01 | 1 | Communication protocol v1 |
| 0x0A | 1 | Power Requirement | 0x00 | Self-powered | USB-C powered, no V_BUS needed |
| 0x0B | 2 | Max Current (mA) | 0x01F4 | 500 mA | Maximum draw (little-endian) |
| 0x0D | 2 | Min Voltage (mV) | 0x1194 | 4500 mV | Minimum supply voltage (little-endian) |
| 0x0F | 2 | Boot Delay (ms) | 0x0064 | 100 ms | Power-on delay (little-endian) |
| 0x11 | 4 | Capability Bitmask | 0x0000000D | — | Bits 0, 2, 3 enabled (see below) |
| 0x15 | 16 | Module Name | 44 65 73 6B... | "Desktop" | ASCII, null-terminated |
| 0x25 | 16 | Serial Number | 44 43 2D 30... | "DC-001-ABCD" | ASCII, null-terminated |
| 0x35 | 198 | User Data | — | — | Available for custom data |
| 0xFB | 4 | CRC32 | 0xXXXXXXXX | — | CRC32 of bytes 0x00–0xFA |

#### Capability Bitmask

**Value:** `0x0000000D` (Bits 0, 2, and 3 set)

| Bit | Value | Capability | Status |
| :---: | :---: | :--- | :--- |
| 0 | 0x01 | Needs Calibration | ✅ Enabled |
| 1 | 0x02 | Requires ACK | ❌ Disabled |
| 2 | 0x04 | Async Data | ✅ Enabled |
| 3 | 0x08 | Supports Sleep | ✅ Enabled |
| 4 | 0x10 | Has Emergency Stop | ❌ Disabled |
| 5 | 0x20 | Supports OTA | ❌ Disabled |
| 6 | 0x40 | Has Battery Backup | ❌ Disabled |
| 7 | 0x80 | Requires Bootloader Mode | ❌ Disabled |
| 8–31 | — | Reserved | — |

### User Interface & IO

- **SK9822-EC20** (D2) — Addressable RGB LED for status or mood lighting.
- **Tactile Switches** (SW1, SW3) — Reset and bootmode selection switching.
- **Snooze Button** (SW2) — Sleep mode activation.
- **USB-C Port** (J1) — For independent programming and powering of the RP2040.

## Integration

This module identifies itself as an environmental sensor class. The ESP-HI-PLUS main board can read data from the onboard sensors through commands to the RP2040 via UART through the magnetic pogo-pin connectors.

### Pinouts (Magnetic Pogo Connectors)

The Desktop module uses two 5-pin magnetic pogo-pin connectors to interface with the main board.

#### J2 (MAG1)

| Pin | Name   | Description                                           |
| --- | ------ | ----------------------------------------------------- |
| 1   | VMAIN  | 2-way power bus shared with main module               |
| 2   | GND    | Ground                                                |
| 3   | UART RX| From main module for data transmission                |
| 4   | UART TX| From main module for data transmission                |
| 5   | EN     | Main module control to enable/disable submodule power |

#### J3 (MAG2)

| Pin | Name     | Description                                     |
| --- | -------- | ----------------------------------------------- |
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
