# Hzbugger

Hzbugger is an open-source, Black Magic Probe (BMP) compatible SWD/JTAG hardware debugger and programmer based on the popular STM32F103 microcontroller. It is designed to act as a convenient, all-in-one USB tool for programming and debugging ARM Cortex-M targets.

## Features

- **Microcontroller**: STM32F103C8T6 (ARM Cortex-M3, 72 MHz)
- **USB Interface**: Direct-plug USB Type-A connector (HX AM 90D)
- **Protection**: On-board SRV05-4 USB ESD protection and 1A fuse
- **Power**: 3.3V logic level with an integrated AMS1117-3.3 linear regulator
- **Auxiliary Interfaces**: 
  - 6-pin GPIO/UART header (J6) for target serial console
  - 4-pin SWD header (J4) for initially flashing the debugger's own firmware
- **User Controls**: Boot0 (SW2) and Reset (SW1) tactile buttons to easily enter system bootloader mode
- **Indicators**: Dedicated Power LED and multiple status LEDs for activity monitoring

## Hardware Repository Structure

This repository contains the full KiCad electronic design automation (EDA) project files:
- `hzbugger.kicad_pro` / `hzbugger.kicad_sch` / `hzbugger.kicad_pcb`: Core KiCad 8.x project files.
- `gerbers/`: Pre-generated Gerber files, drill files, and pick-and-place files for PCB manufacturing.
- `production/`: IPC netlists, unified BOM (`bom.csv`), and positional files for automated assembly.
- `hzbugger.pretty/` & `hzbugger.3dshapes/`: Project-specific footprints and 3D models.

## Bill of Materials (BOM) Highlights

| Reference | Value | Description |
|-----------|-------|-------------|
| U1 | STM32F103C8T6 | Main MCU |
| U2 | SRV05-4 | USB ESD Protection |
| U3 | AMS1117-3.3 | 3.3V LDO Voltage Regulator |
| U4 | HX AM 90D | USB-A Male Connector |
| F1 | 1A | Replaceable/Poly Fuse |
| Y1 | 8MHz | System Crystal |
