# hzbugger

Hzbugger is an STM32F103C8T6-based Black Magic Probe compatible SWD debugger with UART passthrough.

## Hardware summary (rev.A)

- MCU: **STM32F103C8T6**
- USB: onboard USB-A connector
- Main target/debug connector: **J2 (2x5 IDC, 2.54 mm)**
- Onboard programming header: **J4 (1x4)**
- Auxiliary breakout header: **J6 (1x6, GPIO/PB3/4/5)**

## Connectors and pinout

### J2 (IDC 2x5) — target/debug interface

This is the main connector for target SWD + UART.

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

Silkscreen also marks `pin1` and prints odd/even column signal names.

### J4 (1x4, labeled SWD) — programming/debug of Hzbugger MCU

| Pin | Signal | MCU pin |
|---|---|---|
| 1 | GND | GND |
| 2 | +3V3 | +3V3 |
| 3 | DCLK | PA14 |
| 4 | DIO | PA13 |

### J6 (1x6, labeled GPIO)

| Pin | Signal |
|---|---|
| 1 | GND |
| 2 | PB3 / SPI1_SCK (via 22R) |
| 3 | PB4 / SPI1_MISO (via 22R) |
| 4 | PB5 / SPI1_MOSI (via 22R) |
| 5 | +3V3 |
| 6 | GND |

## Power notes

- USB VBUS is routed to **+5V**.
- **AMS1117-3.3** generates **+3V3** from +5V.
- On J2, **both +3V3 (pin 8) and +5V (pin 9) are present**.
- There is no on-board 3.3V/5V output selector switch; choose the target supply rail in your cable/target wiring.

## Firmware adaptation guide

See `blackmagic-probe-adaptation-guide.md` for firmware-porting and usage details.
