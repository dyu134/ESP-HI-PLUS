# ESP-HI-PLUS Sub-modules

This directory contains various snap-on expansion boards for the **ESP-HI-PLUS** main board. These modules connect via the two 5-pin magnetic pogo-pin connectors and are automatically detected by the main board.

## List of Sub-modules

- **[Desktop](./Desktop/README.md)** — Desktop alarm clock and environment monitor. Features temperature/humidity sensing and presence detection.

## Sub-Module Identification

The two 5-pin magnetic pogo-pin connectors (J3, J4) carry an analog ID line that lets the main board auto-detect **what class of sub-module** is attached before initializing it.

### How it works

1. **On the main board (ESP-HI-PLUS)**, a resistor divider is built around R28 (47 kΩ) and R29 (47 kΩ) tied to the 3.3 V rail and GND. With no sub-module attached, the divider midpoint sits at ~1.65 V (mid-supply) and is fed into an ESP32-S3 ADC channel.
2. **Before initializing any sub-board**, firmware reads that ADC voltage.
3. **On the sub-module**, an additional pull-up resistor `R_ID` is connected between the **AUX1/ID0** line and 3.3 V. The parallel combination with R28 shifts the divider voltage to a value that uniquely identifies the sub-module class.
4. The measured ADC voltage is matched against the table below to select the correct driver/configuration for the attached module.
5. **Once the class is known**, the main board reads the **I²C EEPROM** sitting on the sub-module through the shared I²C bus on the pogo connector. The EEPROM carries module-specific metadata (see below) that the firmware uses to finish initializing that particular sub-board variant correctly.

The two-stage lookup (cheap analog ID + rich digital metadata) lets many different sub-modules share the same physical connector and pinout while still being recognized, configured, and updated individually.

### Class Table

| Class | R_ID (sub-module pull-up) | ADC Voltage (V) | Class Type                |
|------:|--------------------------:|----------------:|---------------------------|
| 0     | Open (no module)          | 1.65            | No module attached        |
| 1     | 953 kΩ                    | 1.63            | Actuator — Servo          |
| 2     | 549 kΩ                    | 1.58            | Actuator — Stepper        |
| 3     | 383 kΩ                    | 1.54            | Sensor — Environmental    |
| 4     | 287 kΩ                    | 1.49            | Sensor — Mechanical       |
| 5     | 226 kΩ                    | 1.43            | Audio — Speaker           |
| 6     | 182 kΩ                    | 1.37            | Audio — Microphone        |
| 7     | 154 kΩ                    | 1.31            | Display — Screen          |
| 8     | 130 kΩ                    | 1.24            | Display — LED             |
| 9     | 110 kΩ                    | 1.17            | Communication — 2.4 G     |
| 10    | 95.3 kΩ                   | 1.11            | Communication — LoRA      |
| 11    | 82.5 kΩ                   | 1.04            | Power — Battery           |
| 12    | 71.5 kΩ                   | 0.97            | Power — Solar             |
| 13    | 61.9 kΩ                   | 0.89            | User Defined 1            |
| 14    | 53.6 kΩ                   | 0.82            | User Defined 2            |
| 15    | 47.5 kΩ                   | 0.75            | User Defined 3            |
| 16–31 | —                         | —               | Reserved                  |

Designing your own sub-module? Pick any standard E24 / E96 `R_ID` value from the table and add it from the ID line to 3.3 V on the sub-board — the main board will detect the class automatically on the next power-up.

### EEPROM metadata

The sub-module's I²C EEPROM exposes at minimum the following fields, which the main board reads via standard I²C reads (typically at the address indicated by the detected class, default `0x50` if unspecified):

| Offset | Size | Field             | Description                                                                          |
|-------:|-----:|-------------------|--------------------------------------------------------------------------------------|
| 0x00   | 4    | Magic Number      | ASCII `"ESPM"` (0x45 0x53 0x50 0x4D) — sanity-check that an ESP-HI EEPROM is present |
| 0x04   | 1    | Class ID          | Module-class number matching the class table above (little-endian `u8`)              |
| 0x05   | 1    | Module ID         | Module-class number matching the class table above (little-endian `u8`)              |
| 0x06   | 1    | HW revision       | Sub-board hardware revision (little-endian `u8`)                                     |
| 0x07   | 2    | FW revision       | Sub-board firmware revision (major.minor) (little-endian `u16`)                      |
| 0x09   | 1    | Protocol revision | Communication protocol revision (little-endian `u8`)                                 |
| 0x0A   | 1    | Power requirement | 0x00 = Self-powered, 0x01 = Requires 5V, 0x02 = Can charge                           |
| 0x0B   | 2    | Max Current (mA)  | Maximum current draw (little-endian `u16`), default 500 mA                           |
| 0x0D   | 2    | Min Voltage (mV)  | Minimum supply voltage (little-endian `u16`), default 4500 mV                        |
| 0x0F   | 2    | Boot Delay        | Number of milliseconds to delay after power-on (little-endian `u16`), default 100ms  |
| 0x11   | 4    | Capability Bitmask| Bitmask of capabilities defined below (little-endian `u32`)                          |
| 0x15   | 16   | Module Name       | ASCII string (little-endian `u32`)                                                   |
| 0x25   | 16   | Serial Number     | ASCII string (little-endian `u32`)                                                   |
| 0x35   | 198  | User Data         | ASCII string (little-endian `u32`)                                                   |
| 0xFB   | 4    | CRC32             | CRC32 of the rest of the EEPROM data (little-endian `u32`)                           |

If the magic bytes don't match or the CRC fails, the main board treats the EEPROM as missing/corrupt and falls back to the **class-only driver** selected from the table — letting older or unprogrammed sub-modules still function.

### Capability Bitmask

The 32-bit field at EEPROM offset `0x11` advertises the runtime capabilities of the attached sub-module. Each bit is enumerated below; **bits 8–31 are reserved** for future use and must be written as `0` by sub-module firmware.

| Bit | Name                        | Meaning                                                                                          |
|----:|-----------------------------|--------------------------------------------------------------------------------------------------|
| 0   | Needs Calibration           | Main module must send `CMD_CALIBRATE` before normal ops                                          |
| 1   | Requires ACK                | Main module must wait for `ACK` after every command                                              |
| 2   | Asynchronous Data           | Module sends unsolicited status reports                                                          |
| 3   | Supports Sleep              | Module can enter sleep mode                                                                      |
| 4   | Has Emergency Stop          | Module can trigger an emergency stop                                                             |
| 5   | Supports OTA                | Module can receive firmware updates from main                                                    |
| 6   | Has Battery Backup          | Module has its own backup battery                                                                |
| 7   | Requires Bootloader Mode    | Main must enter bootloader mode before firmware update                                           |
| 8–31| Reserved                    | Reserved — must be `0`                                                                           |
