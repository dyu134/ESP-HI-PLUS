# ESP-HI-PLUS

A compact, feature-rich development board built around the **ESP32-S3-WROOM-1**, designed for handheld and wearable applications combining motion sensing, audio I/O, a small display, and USB-C power delivery with on-board Li-ion battery charging.

![Board](ESP-HI-PLUS.kicad_pcb)

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

- **[0.71" LCD header](https://www.waveshare.com/2inch-LCD-Module.htm)** (J2) — small monochrome or color display.
- **Two SKRPACE010 tactile pushbuttons** (SW1, SW2).
- **JS102011SAQN SPDT slide switch** (SW3).
- **SK9822-EC20** addressable RGB LED (D3), plus discrete status LEDs (D2 orange, D4 red).
- **Two PR5L4015-5P-C-F expansion headers** (J3, J4) — 5-pin 2.54 mm for I/O breakouts.

### Mounting

- Four M2 slim mounting holes (H1–H4).

## Repository Layout

```text
.
├── ESP-HI-PLUS.kicad_pro       # KiCad project file
├── ESP-HI-PLUS.kicad_sch       # Schematic
├── ESP-HI-PLUS.kicad_pcb       # PCB layout
├── fp-lib-table                # Footprint library table
├── sym-lib-table               # Symbol library table
├── fabrication-toolkit-options.json
├── ESP-HI-PLUS Symbols/        # Project-local schematic symbols
├── ESP-HI-PLUS Footprints/     # Project-local footprint libraries (*.pretty)
├── ESP-HI-PLUS 3D Models/      # STEP models used by the 3D viewer
├── ESP-HI-PLUS-backups/        # Auto-generated backups (ignored)
├── production/                 # Manufacturing outputs
│   ├── bom.csv                 # Bill of materials
│   ├── designators.csv         # Reference designators
│   ├── positions.csv           # Pick-and-place positions
│   ├── netlist.ipc             # IPC-D-356 netlist
│   └── ESP-HI-PLUS.zip         # Production archive
└── .history/                   # KiCad local history (ignored)
```

## Bill of Materials (summary)

| Refs        | Description                                                             |
|-------------|-------------------------------------------------------------------------|
| U2          | ESP32-S3-WROOM-1 module                                                 |
| U3          | TLV75733PDBV — 3.3 V LDO                                                |
| U4          | TP4065 — Li-ion charger                                                 |
| U5          | DMM-4026-B I2S microphone                                               |
| U7          | MAX98357A — I2S Class-D amplifier                                       |
| U8          | LSM6DSV16X — 6-axis IMU                                                 |
| U1, U6      | MAX40200AUK — ideal diode                                               |
| D1/D7/D8    | USBLC6-2P6 — USB ESD TVS array                                          |
| D3          | SK9822-EC20 — addressable RGB LED                                       |
| J1          | USB-C receptacle                                                        |
| J2          | [2" IPS LCD connector](https://www.waveshare.com/2inch-LCD-Module.htm)  |
| SW1, SW2    | SKRPACE010 tactile pushbuttons                                          |
| SW3         | JS102011SAQN SPDT slide switch                                          |

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
