# Portable Sub-module

The **Portable** sub-module is a high-performance expansion board for the **ESP-HI-PLUS**, designed for mobile use and advanced user interaction. Powered by an STM32U5 series MCU, it provides significant local processing power and support for rich graphical interfaces.

*(Space for 3D Model Image)*

## Features

### Core Compute

- **STM32U5G9VJTxQ** (U1) — High-performance, ultra-low-power ARM Cortex-M33 microcontroller with advanced graphics support and 4 MB of Flash / 3 MB of RAM.

### Power Management

- **TLV75733PDBV** (U4) — 3.3V Low-Dropout (LDO) regulator providing stable power to the MCU and peripherals.
- **TP4065** (U3) — Standalone linear Li-Ion battery charger with thermal regulation.
- **TCPP03-M20** (U2) — USB-PD 3.1 controller with protection for dual-role power (DRP) and over-voltage/current protection.

### Peripherals & IO

- *(Space for Display/LCD additions — DCMI and SPI interfaces available)*
- *(Space for Audio additions — SAI interfaces available)*
- *(Space for Sensor additions)*

### Identification & Metadata

The Portable module follows the standard [ESP-HI-PLUS identification protocol](../README.md).

- **R_ID (383 kΩ)** — Currently identifies the module as **Class 3 (Sensor — Environmental)**. *Note: This may be updated to Class 11 (Power — Battery) or Class 7 (Display) in future revisions.*
- **AT24CS02** (U10) — 2 Kbit I²C EEPROM carrying module metadata and serial number.

#### EEPROM Metadata

| Offset | Size | Field | Value (Hex) | Value (Decoded) | Description |
| :------: | :----: | :------ | :-----------: | :---------------- | :------------ |
| 0x00 | 4 | Magic Number | 45 53 50 4D | "ESPM" | ASCII sanity check |
| 0x04 | 1 | Class ID | 0x03 | 3 | Sensor - Environmental |
| 0x05 | 1 | Module ID | 0x02 | 2 | Portable v1 |
| 0x06 | 1 | HW Revision | 0x01 | 1 | First PCB revision |
| 0x07 | 2 | FW Revision | 0x0100 | 1.0 | Major.Minor |
| 0x09 | 1 | Protocol Revision | 0x01 | 1 | Communication protocol v1 |
| 0x0A | 1 | Power Requirement | 0x02 | Can charge | (Assumed for Portable) |
| 0x0B | 2 | Max Current (mA) | 0x01F4 | 500 mA | Maximum draw (little-endian) |
| 0x0D | 2 | Min Voltage (mV) | 0x1194 | 4500 mV | Minimum supply voltage (little-endian) |
| 0x0F | 2 | Boot Delay (ms) | 0x0064 | 100 ms | Power-on delay (little-endian) |
| 0x11 | 4 | Capability Bitmask | 0x00000000 | — | TBD |
| 0x15 | 16 | Module Name | 50 6F 72 74... | "Portable" | ASCII, null-terminated |
| 0x25 | 16 | Serial Number | 50 52 2D 30... | "PR-001-XXXX" | ASCII, null-terminated |
| 0x35 | 198 | User Data | — | — | Available for custom data |
| 0xFB | 4 | CRC32 | 0xXXXXXXXX | — | CRC32 of bytes 0x00–0xFA |

#### Capability Bitmask (Proposed)

**Value:** `0x00000000` (TBD)

| Bit | Value | Capability | Status |
| :---: | :---: | :--- | :--- |
| 0 | 0x01 | Needs Calibration | ❌ Disabled |
| 1 | 0x02 | Requires ACK | ✅ Enabled |
| 2 | 0x04 | Async Data | ✅ Enabled |
| 3 | 0x08 | Supports Sleep | ✅ Enabled |
| 4 | 0x10 | Has Emergency Stop | ❌ Disabled |
| 5 | 0x20 | Supports OTA | ✅ Enabled |
| 6 | 0x40 | Has Battery Backup | ✅ Enabled |
| 7 | 0x80 | Requires Bootloader Mode | ❌ Disabled |

## Integration

The Portable module uses the high-performance STM32U5 MCU to handle complex tasks locally. It communicates with the ESP-HI-PLUS main board via UART and I²C.

### Pinouts (Magnetic Pogo Connectors)

#### J2 (MAG1)

| Pin | Name   | Description                                           |
| --- | ------ | ----------------------------------------------------- |
| 1   | VMAIN  | 2-way power bus shared with main module               |
| 2   | GND    | Ground                                                |
| 3   | UART RX| Data from main module                                 |
| 4   | UART TX| Data to main module                                   |
| 5   | EN     | Main module control to enable/disable submodule power |

#### J3 (MAG2)

| Pin | Name     | Description                                     |
| --- | -------- | ----------------------------------------------- |
| 1   | I2C SDA  | For reading module ID metadata EEPROM           |
| 2   | I2C SCL  | For reading module ID metadata EEPROM           |
| 3   | AUX1/ID0 | Hardware class identifier resistor              |
| 4   | AUX2     | Auxiliary data pin                              |
| 5   | AUX3     | Auxiliary data pin                              |

## Repository Layout

```text
.
├── Portable.kicad_pro           # KiCad project file
├── Portable.kicad_sch           # Schematic
├── Portable.kicad_pcb           # PCB layout (Initial)
├── Portable-backups/            # Project backups
└── README.md                    # This file
```
