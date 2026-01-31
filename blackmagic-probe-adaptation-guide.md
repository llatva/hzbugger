# Black Magic Probe Firmware Adaptation Guide for Hzbugger

## Overview

Hzbugger is a Black Magic Probe (BMP)-compatible SWD debugger. This guide walks through adapting upstream BMP firmware to the Hzbugger hardware, highlights the key firmware features, and provides concrete examples for using both GDB debugging and the built-in UART passthrough. The goal is to help you build a firmware image that exposes:

- **Simultaneous SWD + UART** using two USB CDC ACM serial ports.
- **GDB server integration** over the primary USB serial port.
- **Generic serial console access** over the secondary USB serial port.
- **Target power output** support at **3.3 V or 5 V** (selectable on the board).

> Note: This guide assumes you are familiar with the upstream Black Magic Probe firmware structure. When in doubt, refer to the upstream BMP documentation for the canonical build steps.

## Prerequisites

- The upstream Black Magic Probe firmware source.
- A toolchain such as `arm-none-eabi-gcc`.
- USB access to the Hzbugger device for flashing.
- The Hzbugger schematic or silk labels for pin and voltage selection reference.

## 1) Identify the Hzbugger Hardware Mapping

Before changing firmware, map the Hzbugger pins and peripherals to the BMP firmware expectations:

- **SWD interface:** SWDIO, SWCLK, and GND.
- **UART passthrough:** UART TX/RX and GND.
- **USB device:** VID/PID, product strings, and USB endpoints.
- **Power output:** board selector/jumper that toggles 3.3 V vs 5 V output.

Document these pins in a short table (even a private note) so you can mirror them in the firmware definitions.

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

## 4) STM32F103 DFU Bootloader (Firmware Upgrade Path)

Hzbugger uses an **STM32F103** MCU, which includes a built-in USB DFU bootloader in system memory. You can leverage this for easy firmware upgrades without external tools.

### Entering DFU Mode (STM32F103)

The STM32F103 enters its ROM DFU bootloader when **BOOT0 = 1** and **BOOT1 = 0** at reset:

- **BOOT0 high:** pull-up to 3.3 V (via jumper or switch).
- **BOOT1 low:** BOOT1 is tied to PB2; keep it pulled down (default).

Recommended hardware provisions for development boards:

- A **BOOT0 jumper/switch** so DFU mode can be entered without rewiring.
- A **RESET button** to re-assert reset after changing BOOT0.
- Keep **BOOT1 (PB2) hard-tied low** unless you specifically need alternate boot modes.

After setting BOOT0 high, reset the MCU and it should enumerate as a USB DFU device.

### Flashing via DFU

On Linux (with `dfu-util` installed):

```bash
dfu-util -l
dfu-util -a 0 -s 0x08000000:leave -D blackmagic.bin
```

On Windows/macOS, use the **STM32CubeProgrammer** GUI to connect to the DFU device and program the binary at address `0x08000000`.

Once flashing is complete, return **BOOT0 low** and reset to run the new firmware.

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

## 9) Target Power Output (3.3 V / 5 V)

Hzbugger can supply **either 3.3 V or 5 V** to the target device.

- Locate the **voltage selection jumper or solder bridge** on the board.
- Set it to match your target’s required voltage.
- Ensure that the target is not simultaneously powered from another source unless the design explicitly supports it.

> Always verify the silk-screen or schematic for the correct selector position before powering the target.

## 10) Quick Troubleshooting Checklist

- **No USB ports appear:** verify firmware is flashed and USB cable is data-capable.
- **GDB fails to connect:** check SWD wiring and common ground, then run `monitor swdp_scan`.
- **UART has no output:** verify target baud rate and RX/TX pin orientation.
- **DFU mode not detected:** confirm BOOT0 is high, BOOT1 is low, and reset was applied.
- **Unstable target behavior:** confirm correct 3.3 V / 5 V selection and avoid double power sources.

## Summary

By defining a minimal BMP platform for Hzbugger, you can unlock full SWD debugging alongside real-time UART logging. With dual CDC ACM ports and selectable target power, Hzbugger becomes a compact, flexible debugging tool for modern ARM microcontrollers.
