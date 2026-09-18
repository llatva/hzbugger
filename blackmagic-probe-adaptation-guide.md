# Black Magic Probe Firmware Adaptation Guide for Hzbugger

## Overview

Hzbugger is a Black Magic Probe (BMP)-compatible SWD debugger. This guide walks through adapting upstream BMP firmware to the Hzbugger hardware, highlights the key firmware features, and provides concrete examples for using both GDB debugging and the built-in UART passthrough. The goal is to help you build a firmware image that exposes:

- **Simultaneous SWD + UART** using two USB CDC ACM serial ports.
- **GDB server integration** over the primary USB serial port.
- **Generic serial console access** over the secondary USB serial port.
- **Target connection over IDC 2x5** with SWD, UART, reset, and power rails.

> Note: This guide assumes you are familiar with the upstream Black Magic Probe firmware structure. When in doubt, refer to the upstream BMP documentation for the canonical build steps.

## Prerequisites

- The upstream Black Magic Probe firmware source.
- A toolchain such as `arm-none-eabi-gcc`.
- USB access to the Hzbugger device for flashing.
- The Hzbugger schematic or silk labels for pin and power-rail reference.

## 1) Identify the Hzbugger Hardware Mapping

Before changing firmware, map the Hzbugger pins and peripherals to BMP firmware expectations.

### Main target connector: J2 (IDC 2x5, 2.54 mm)

Use this as the canonical target pinout:

| Pin | Signal |
|---|---|
| 1 | UART_TX |
| 2 | UART_RX |
| 3 | SWO_TRACE |
| 4 | RST |
| 5 | GND |
| 6 | SWDIO |
| 7 | SWCLK |
| 8 | +3V3 |
| 9 | +5V |
| 10 | GND |

### Onboard programming connector: J4 (1x4, labeled SWD)

This header is for programming/debugging the Hzbugger MCU itself:

| Pin | Signal | MCU pin |
|---|---|---|
| 1 | GND | GND |
| 2 | +3V3 | +3V3 |
| 3 | DCLK | PA14 |
| 4 | DIO | PA13 |

### Auxiliary GPIO connector: J6 (1x6)

| Pin | Signal |
|---|---|
| 1 | GND |
| 2 | PB3 / SPI1_SCK (via 22R) |
| 3 | PB4 / SPI1_MISO (via 22R) |
| 4 | PB5 / SPI1_MOSI (via 22R) |
| 5 | +3V3 |
| 6 | GND |

### Power behavior on rev.A

- USB VBUS provides the board +5V rail.
- AMS1117-3.3 generates +3V3.
- J2 exposes both +3V3 (pin 8) and +5V (pin 9) simultaneously.
- There is no onboard selector that switches target output between 3.3V and 5V.

## 2) Create a New BMP Platform Definition

In the BMP firmware tree, create a new platform target (or copy the closest existing one) so the build system produces firmware tailored to Hzbugger.

Typical steps include:

1. **Add a platform directory** (e.g., `platforms/hzbugger/`).
2. **Define GPIO pin mappings** for SWDIO, SWCLK, UART TX/RX, LEDs, and reset.
3. **Set USB descriptors** (product name like `Hzbugger` is useful for users).
4. **Enable dual CDC ACM ports** to expose both GDB and UART.

Keep the changes minimal: only update what is necessary for the Hzbugger pinout and USB identification.

## 3) Build and Flash the Firmware

Once the platform definition is in place, build the firmware using the normal BMP workflow. A typical sequence looks like this:

```bash
make clean
make PROBE_HOST=hzbugger
```

Flash the resulting binary to the Hzbugger using your preferred method (DFU, SWD, or a board-specific bootloader).

## 4) STM32F103 DFU Bootloader Notes

Hzbugger uses an **STM32F103** MCU, which has a built-in ROM DFU bootloader. However, on this board revision, BOOT0 is not routed to a dedicated external jumper/switch.

### Entering DFU Mode on this board

ROM DFU still requires **BOOT0 = 1** and reset, but rev.A does not provide a dedicated BOOT0 user control. In practice:

- The default build is intended to be flashed/debugged over SWD (J4/J2).
- DFU on this hardware requires board-level rework or temporary BOOT0 injection.
- BOOT1 (PB2) remains tied low by default, which is the normal configuration.

### If DFU is available in your setup

When BOOT0 is externally forced high and the device enumerates in DFU mode, flash as usual (Linux example):

```bash
dfu-util -l
dfu-util -a 0 -s 0x08000000:leave -D blackmagic.bin
```

On Windows/macOS, use **STM32CubeProgrammer** to program at `0x08000000`.

After flashing, restore normal boot configuration and reset.

## 5) Verify USB Enumeration

When the firmware is running, the Hzbugger should enumerate as **two serial devices**:

- **Primary CDC port:** BMP GDB server.
- **Secondary CDC port:** UART passthrough.

On Linux, you might see:

- `/dev/ttyACM0` (GDB)
- `/dev/ttyACM1` (UART)

> The exact numbering may vary depending on the system. Identify by unplugging/replugging or checking `dmesg` output.

## 6) Using GDB Over SWD (Primary Serial Port)

Connect GDB to the probe using the primary port. Example workflow:

```bash
arm-none-eabi-gdb build/firmware.elf
```

In the GDB session:

```gdb
target extended-remote /dev/ttyACM0
monitor swdp_scan
attach 1
load
continue
```

### Common GDB Commands

- `monitor swdp_scan` — list available SWD targets.
- `attach <n>` — attach to target number `n`.
- `load` — program the target.
- `reset` — reset the target after loading.

This gives a full-featured debugging experience (breakpoints, memory inspection, etc.).

## 7) Using UART Passthrough (Secondary Serial Port)

The second CDC port is a transparent serial bridge to the target MCU UART pins. Use it as a normal serial terminal:

```bash
screen /dev/ttyACM1 115200
```

Or with `picocom`:

```bash
picocom -b 115200 /dev/ttyACM1
```

This port can be used for logging, CLI shells, or any debug printf output from the target.

## 8) Simultaneous SWD + UART Usage

Hzbugger supports **simultaneous SWD debugging and UART logging**. Typical flow:

1. Connect GDB to the primary port (`/dev/ttyACM0`).
2. Start a serial terminal on the secondary port (`/dev/ttyACM1`).
3. Debug while streaming logs in real time.

This is especially useful for correlating firmware logs with breakpoints or crash reproduction.

## 9) Target Power Output (+3V3 / +5V rails on J2)

Hzbugger exposes **both +3V3 and +5V rails** on J2:

- **J2 pin 8:** +3V3
- **J2 pin 9:** +5V

There is no onboard selector that switches one rail to the other. Choose the correct rail in your cable/target-side wiring and avoid back-powering conflicts.

> Always verify target voltage requirements before connecting either power rail.

## 10) Quick Troubleshooting Checklist

- **No USB ports appear:** verify firmware is flashed and USB cable is data-capable.
- **GDB fails to connect:** check SWD wiring and common ground, then run `monitor swdp_scan`.
- **UART has no output:** verify target baud rate and RX/TX pin orientation.
- **DFU mode not detected:** confirm BOOT0 is high, BOOT1 is low, and reset was applied.
- **Unstable target behavior:** verify you are using the intended rail (+3V3 or +5V) and avoid double power sources.

## Summary

By defining a minimal BMP platform for Hzbugger, you can unlock full SWD debugging alongside real-time UART logging. With dual CDC ACM ports and explicit +3V3/+5V target rail pins, Hzbugger becomes a compact, flexible debugging tool for modern ARM microcontrollers.
