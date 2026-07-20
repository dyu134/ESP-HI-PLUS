# ESP-HI-PLUS Sub-modules

This directory contains various snap-on expansion boards for the **ESP-HI-PLUS** main board. These modules connect via the two 5-pin magnetic pogo-pin connectors and are automatically detected by the main board.

## List of Sub-modules

- **[Desktop](./Desktop/README.md)** — Desktop alarm clock and environment monitor. Features temperature/humidity sensing and presence detection.

## Designing a Sub-module

To design a new sub-module:

1. Select a class from the Class Table in the [main README](../README.md#class-table).
2. Add the corresponding `R_ID` pull-up resistor to the **AUX1/ID0** line.
3. Include an I²C EEPROM for metadata identification.
4. Follow the mechanical footprint for the **5-pin magnetic pogo-pin connector(s)**. Modules can use one or both connectors (MAG1 and MAG2).
