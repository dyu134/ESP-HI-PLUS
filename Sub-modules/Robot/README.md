# Robot Sub-module

The **Robot** sub-module is a high-performance actuator and sensing expansion board for the **ESP-HI-PLUS**, designed to transform the device into a mobile or interactive robotic platform. Powered by an RP2040 MCU, it handles real-time motor control, battery management, and inertial sensing.

*(Space for 3D Model Image)*

## Features

### Core Compute

- **RP2040** (U3) — Dual-core ARM Cortex-M0+ microcontroller for real-time servo PWM generation and sensor processing.
- **16 Mbit Flash** (U?) — Storage for RP2040 firmware.

### Actuation & Sensing

- **Servo Support** (M1–M4) — High-current outputs for up to 4 high-torque servo motors (e.g., MG996R).
- **LSM6DSV16X** (U8) — 6-axis IMU with embedded sensor fusion and Machine Learning Core for orientation and motion detection.
- **INA3221** (U11) — Triple-channel current and voltage monitor for real-time tracking of servo and system power consumption.

### Power Management

- **TLV75733PDBV** (U4) — 3.3V Low-Dropout (LDO) regulator providing stable power to the MCU and peripherals.
- **MAX17049** (U5) — 2-cell Li-ion battery fuel gauge with ModelGauge m5 algorithm for accurate state-of-charge reporting.
- **MAX40200AUK** (U2) — Ultra-low voltage drop ideal diode for efficient power path management.
- **UBEC Input** — Support for external 5V UBEC to power high-current servos from the battery.
- **XT60 Connector** (J4) — High-current battery interface (typically 2S Li-ion).

### User Interface & IO

- **SK9822-EC20** (D6) — Addressable RGB LED for status or mood lighting.
- **SKRPACE010** (SW3, SW?) — Tactile switches for reset, boot mode, or user interaction.

### Identification & Metadata

The Robot module follows the standard [ESP-HI-PLUS identification protocol](../README.md).

- **R_ID (953 kΩ)** — Identifies the module as **Class 1 (Actuator — Servo)**.
- **AT24CS02** (U10) — 2 Kbit I²C EEPROM carrying module metadata and serial number.

#### EEPROM Metadata

| Offset | Size | Field | Value (Hex) | Value (Decoded) | Description |
| :------: | :----: | :------ | :-----------: | :---------------- | :------------ |
| 0x00 | 4 | Magic Number | 45 53 50 4D | "ESPM" | ASCII sanity check |
| 0x04 | 1 | Class ID | 0x01 | 1 | Actuator - Servo |
| 0x05 | 1 | Module ID | 0x03 | 3 | Robot v1 |
| 0x06 | 1 | HW Revision | 0x01 | 1 | First PCB revision |
| 0x07 | 2 | FW Revision | 0x0100 | 1.0 | Major.Minor |
| 0x09 | 1 | Protocol Revision | 0x01 | 1 | Communication protocol v1 |
| 0x0A | 1 | Power Requirement | 0x00 | Self-powered | Powered by 2S battery |
| 0x0B | 2 | Max Current (mA) | 0x07D0 | 2000 mA | Maximum draw (little-endian) |
| 0x0D | 2 | Min Voltage (mV) | 0x1770 | 6000 mV | Minimum supply voltage (2S) |
| 0x0F | 2 | Boot Delay (ms) | 0x0064 | 100 ms | Power-on delay (little-endian) |
| 0x11 | 4 | Capability Bitmask | 0x00000014 | — | Bits 2, 4 enabled (Async, E-Stop) |
| 0x15 | 16 | Module Name | 52 6F 62 6F... | "Robot" | ASCII, null-terminated |
| 0x25 | 16 | Serial Number | 52 42 2D 30... | "RB-001-XXXX" | ASCII, null-terminated |
| 0x35 | 198 | User Data | — | — | Available for custom data |
| 0xFB | 4 | CRC32 | 0xXXXXXXXX | — | CRC32 of bytes 0x00–0xFA |

#### Capability Bitmask (Proposed)

**Value:** `0x00000014`

| Bit | Value | Capability | Status |
| :---: | :---: | :--- | :--- |
| 0 | 0x01 | Needs Calibration | ❌ Disabled |
| 1 | 0x02 | Requires ACK | ❌ Disabled |
| 2 | 0x04 | Async Data | ✅ Enabled |
| 3 | 0x08 | Supports Sleep | ❌ Disabled |
| 4 | 0x10 | Has Emergency Stop | ✅ Enabled |
| 5 | 0x20 | Supports OTA | ❌ Disabled |
| 6 | 0x40 | Has Battery Backup | ❌ Disabled |
| 7 | 0x80 | Requires Bootloader Mode | ❌ Disabled |

## Integration

The Robot module uses an RP2040 to handle low-level PWM and sensor data. It communicates with the ESP-HI-PLUS main board via UART and I²C.

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
├── Robot.kicad_pro           # KiCad project file
├── Robot.kicad_sch           # Schematic
├── Robot.kicad_pcb           # PCB layout
├── Robot Symbols/            # Project-local schematic symbols
└── README.md                 # This file
```
