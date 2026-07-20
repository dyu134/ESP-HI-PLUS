# ESP-HI-PLUS

An **AI-enabled voice assistant with on-screen interaction**, built around the **ESP32-S3-WROOM-1**. The board integrates a microphone, speaker amplifier, IMU, 2" IPS LCD, and USB-C with on-board Li-ion battery charging, and exposes **two 5-pin magnetic pogo-pin connectors** for attaching various sub-modules. Designed for handheld and wearable applications that combine on-device conversational AI, audio capture/playback, display feedback, and motion sensing.

![ESP-HI-PLUS 3D Model](ESP-HI-PLUS%203D%20Model%20Image.png)

## Features

### Core Compute

- **ESP32-S3-WROOM-1** (U2) — 32-bit dual-core LX7 MCU with Wi-Fi 4, BLE 5 (LE), and a rich peripheral set including I2S, I2C, SPI, and USB.

### Sensors

- **LSM6DSV16X** 6-axis IMU (U8) — 3-axis accelerometer + 3-axis gyroscope with I3C/I2C/SPI interface for motion tracking.
- **DMM-4026-B I2S Microphone** (U5) — digital MEMS microphone over I2S for audio capture.

### Audio Output

- **MAX98357A** Class-D I2S amplifier (U7) — drives an external speaker (LS1) with no required MCLK.
- **PicoBlade speaker connector** (LS1) for an off-board speaker.

### Power

- **USB-C input** (J1) with **USBLC6-2P6** ESD protection (D1/D7/D8).
- **TP4065** linear Li-ion/Li-Po charger (U4) — single-cell charging from USB.
- **TLV75733PDBV** 3.3 V LDO (U3) — system rail for the MCU and logic.
- **MAX40200AUK** ideal-diode OR-ing (U1, U6) — seamless USB / battery switching with low dropout.
- **PicoBlade battery connector** (BT1) for a single-cell Li-ion / Li-Po pack.
- **PMEG4010CEJ** Schottky diode (D5) and **MF-MSMF200** PTC fuse (F1) for input/battery protection.
- **TGL34-6.8CA** TVS diode (D6) for transient suppression.

### User Interface

- **[2" LCD header](https://www.waveshare.com/2inch-LCD-Module.htm)** (J2) — 2" IPS LCD display.
- **Two SKRPACE010 tactile pushbuttons** (SW1, SW2).
- **JS102011SAQN SPDT slide switch** (SW3).
- **SK9822-EC20** addressable RGB LED (D3), plus discrete status LEDs (D2 orange for battery charging, D4 red for USB power).
- **Two PR5L4015-5P-C-F magnetic pogo-pin connectors** (J3, J4) — 5-pin 2.54 mm headers that mate with **magnetic pogo-pin sub-modules** for snap-on expansion.

### Mounting

- Four M2 slim mounting holes (H1–H4).

## Sub-Module Identification

The two 5-pin magnetic pogo-pin connectors (J3, J4) carry an analog ID line that lets the main board auto-detect **what class of sub-module** is attached before initializing it.

### How it works

1. **On the main board (ESP-HI-PLUS)**, a resistor divider is built around R28 (47 kΩ) and R29 (47 kΩ) tied to the 3.3 V rail and GND. With no sub-module attached, the divider midpoint sits at ~1.65 V (mid-supply) and is fed into an ESP32-S3 ADC channel.
2. **Before initializing any sub-board**, firmware reads that ADC voltage.
3. **On the sub-module**, an additional pull-up resistor `R_ID` is connected between the **AUX1/ID0** line and 3.3 V. The parallel combination with R28 shifts the divider voltage to a value that uniquely identifies the sub-module class.
4. The measured ADC voltage is matched against the table below to select the correct driver/configuration for the attached module.
5. **Once the class is known**, the main board reads the **I²C EEPROM** sitting on the sub-module through the shared I²C bus on the pogo connector. The EEPROM carries module-specific metadata (see below) that the firmware uses to finish initializing that particular sub-board variant correctly.

The two-stage lookup (cheap analog ID + rich digital metadata) lets many different sub-modules share the same physical connector and pinout while still being recognized, configured, and updated individually.

## Available Sub-modules

Detailed information about supported sub-modules can be found in the [Sub-modules directory](./Sub-modules/README.md).

- **[Desktop](./Sub-modules/Desktop/README.md)** — A desktop alarm clock and environment monitor featuring temperature, humidity, and presence sensing.

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

## Repository Layout

```text
.
├── ESP-HI-PLUS.kicad_pro       # KiCad project file
├── ESP-HI-PLUS.kicad_sch       # Schematic
├── ESP-HI-PLUS.kicad_pcb       # PCB layout
├── Sub-modules/                # Snap-on expansion boards
│   └── Desktop/                # Desktop alarm clock / environment monitor
├── fp-lib-table                # Footprint library table
├── sym-lib-table               # Symbol library table
├── fabrication-toolkit-options.json
├── ESP-HI-PLUS Symbols/        # Project-local schematic symbols
├── ESP-HI-PLUS Footprints/     # Project-local footprint libraries (*.pretty)
├── ESP-HI-PLUS 3D Models/      # STEP models used by the 3D viewer
├── production/                 # Manufacturing outputs
    ├── bom.csv                 # Bill of materials
    ├── designators.csv         # Reference designators
    ├── positions.csv           # Pick-and-place positions
    ├── netlist.ipc             # IPC-D-356 netlist
    └── ESP-HI-PLUS.zip         # Production archive
```

## Bill of Materials (summary)

| Refs        | Description                                                                                  |
|-------------|----------------------------------------------------------------------------------------------|
| U2          | ESP32-S3-WROOM-1 module                                                                      |
| U3          | TLV75733PDBV — 3.3 V LDO                                                                     |
| U4          | TP4065 — Li-ion charger                                                                      |
| U5          | DMM-4026-B I2S microphone                                                                    |
| U7          | MAX98357A — I2S Class-D amplifier                                                            |
| U8          | LSM6DSV16X — 6-axis IMU                                                                      |
| U1, U6      | MAX40200AUK — ideal diode                                                                    |
| D1/D7/D8    | USBLC6-2P6 — USB ESD TVS array                                                               |
| D3          | SK9822-EC20 — addressable RGB LED                                                            |
| J1          | USB-C receptacle                                                                             |
| J2          | [2" IPS LCD connector](https://www.waveshare.com/2inch-LCD-Module.htm)                       |
| J3, J4      | Two 5-pin magnetic pogo-pin connectors (PR5L4015-5P-C-F) for sub-modules                     |
| SW1, SW2    | SKRPACE010 tactile pushbuttons                                                               |
| SW3         | JS102011SAQN SPDT slide switch                                                               |
| R28, R29    | 47 kΩ sub-module **AUX1/ID0** detection resistors (0402) — divider used for ADC class lookup |

See `production/bom.csv` for the full BOM with footprints and quantities.

## Getting Started

1. Open `ESP-HI-PLUS.kicad_pro` in KiCad 8 or later.
2. Review the schematic (`ESP-HI-PLUS.kicad_sch`) and PCB (`ESP-HI-PLUS.kicad_pcb`).
3. Re-generate fabrication outputs as needed (Gerbers, drill files, pick-and-place).
4. Flash the ESP32-S3 via the USB-C port using ESP-IDF or Arduino.

## Manufacturing Outputs

The `production/` folder contains ready-to-use outputs:

- **`bom.csv`** — bill of materials for sourcing.
- **`positions.csv`** — pick-and-place rotation/coordinates for SMT assembly.
- **`netlist.ipc`** — IPC-D-356 test netlist.
- **`ESP-HI-PLUS.zip`** — full fabrication archive (Gerbers + drill included).

## License

License not specified. Add a `LICENSE` file if you intend to distribute this design.
