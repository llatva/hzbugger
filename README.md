# hzbugger

Hzbugger is an STM32F103C8T6-based Black Magic Probe compatible SWD debugger with UART passthrough.

Basic functionality works out-of-box with BMP bluepill image. Extra LEDs and AUX header requires fw customization.
See [BlackMagic Probe adaptation guide](./blackmagic-probe-adaptation-guide.md) for firmware-porting and usage details.

<img src="./hzbugger-revA-3D-top.png" width="500"> <img src="./hzbugger-revA-3D-bot.png" width="500">

## Hardware summary (rev.A)

- MCU: **STM32F103C8T6**
- USB: onboard USB-A
- Main target/debug connector: **J2 (2x5 IDC, 2.54 mm)**
- Onboard programming header: **J4 (1x4)**
- Auxiliary breakout header: **J6 (1x6, GPIO/PB3/4/5)**

## References

* Website of the BlackMagic Probe: https://black-magic.org/
* BMP firmware repository: https://codeberg.org/blackmagic-debug/blackmagic

## LICENSE

TBD

